```java
package com.nexalinx.nexadoc.service.impl;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.nexalinx.nexadoc.entity.OTP;
import com.nexalinx.nexadoc.entity.User;
import com.nexalinx.nexadoc.exception.CustomMessagePresentException;
import com.nexalinx.nexadoc.repository.OTPRepository;
import com.nexalinx.nexadoc.repository.UserRepository;
import com.nexalinx.nexadoc.response.SMSResponse;
import com.nexalinx.nexadoc.service.UserService;
import io.micrometer.common.util.StringUtils;
import jakarta.validation.constraints.NotNull;
import lombok.RequiredArgsConstructor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.client.RestClientException;

import java.security.SecureRandom;
import java.util.Optional;

import static com.nexalinx.nexadoc.constant.ValidatorConstants.EXPIRE_HOURS;
import static com.nexalinx.nexadoc.constant.ValidatorConstants.EXPIRE_MINUTES;

@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    private static final Logger logger = LoggerFactory.getLogger(UserServiceImpl.class);
    private static final String OTP_MESSAGE_TEMPLATE = "%s is your One Time Password (OTP).";
    private static final String LOGIN_REQUEST_BODY = "{\"username\":\"admin\",\"password\":\"1234\"}";
    private static final String SMS_REQUEST_BODY_TEMPLATE = "{\"contacts\": [\"%s\"], \"content\": \"%s\", \"language\": \"ENGLISH\"}";
    private static final long MINUTES_TO_MILLIS = 60L * 1000L;
    private static final long HOURS_TO_MILLIS = 60L * 60L * 1000L;
    private static final int MAX_OTP_ATTEMPTS = 3;
    private static final int OTP_UPPER_BOUND = 10000;

    private final UserRepository userRepository;
    private final OTPRepository otpRepository;
    private final RestTemplate restTemplate;
    private final ObjectMapper objectMapper;

    @Value("${api.ami.sms-login-api-url}")
    private String smsLoginApiUrl;

    @Value("${api.ami.sms-send-api-url}")
    private String smsSendApiUrl;

    /**
     * {@inheritDoc}
     */
    @Override
    public Optional<User> findByEmailExist(String email) {
        return userRepository.findByEmail(email);
    }

    @Override
    public boolean validationOtpAPi(String otpCode, @NotNull String firebaseUserId) {
        User user = findByFirebaseUserIdOrThrow(firebaseUserId);
        Optional<OTP> otpOptional = otpRepository.findByOtpCodeAndUser(otpCode, user);

        if (otpOptional.isEmpty()) {
            logger.error("Invalid OTP for user: {}", firebaseUserId);
            throw new CustomMessagePresentException("Invalid OTP");
        }

        OTP otp = otpOptional.get();
        
        if (isOtpExpired(otp.getOtpExpiration())) {
            logger.error("OTP expired for user: {}", firebaseUserId);
            throw new CustomMessagePresentException("OTP has expired");
        }

        if (isValidOtp(otp.getOtpCode(), otpCode)) {
            clearOtpAndVerifyUser(otp, user);
            return true;
        }

        logger.error("Invalid OTP code for user: {}", firebaseUserId);
        throw new CustomMessagePresentException("Invalid OTP");
    }

    @Override
    public boolean regenerateOptAPi(String mobile) {
        validateMobileInput(mobile);
        
        User user = findUserByMobile(mobile);
        validateUserNotAlreadyVerified(user);

        Optional<OTP> otpOptional = otpRepository.findByUser(user);
        
        if (otpOptional.isPresent()) {
            OTP existingOtp = otpOptional.get();
            validateOtpRateLimit(existingOtp);
            updateExistingOtp(existingOtp, mobile);
        } else {
            createNewOtp(user, mobile);
        }

        return true;
    }

    @Override
    public boolean sendOtpAPi(String firebaseUserId) {
        User user = findByFirebaseUserIdOrThrow(firebaseUserId);
        
        if (user.getPhoneNumber() == null) {
            throw new CustomMessagePresentException("Phone number not found for user");
        }

        String otpCode = generateOtpCode();
        
        if (!sendSmsOtp(user.getPhoneNumber(), otpCode)) {
            throw new CustomMessagePresentException("Failed to send OTP");
        }

        saveNewOtp(user, otpCode);
        return true;
    }

    private void validateMobileInput(String mobile) {
        if (StringUtils.isBlank(mobile)) {
            logger.error("Mobile number is required");
            throw new CustomMessagePresentException("Mobile number is required");
        }
    }

    private User findUserByMobile(String mobile) {
        return userRepository.findByPhoneNumber(mobile)
                .orElseThrow(() -> new CustomMessagePresentException("Invalid mobile number"));
    }

    private void validateUserNotAlreadyVerified(User user) {
        if (Boolean.TRUE.equals(user.getIsPhoneNumberVerified())) {
            logger.info("User already verified: {}", user.getPhoneNumber());
            throw new CustomMessagePresentException("User already verified");
        }
    }

    private void validateOtpRateLimit(OTP existingOtp) {
        long currentTime = System.currentTimeMillis();
        
        // Check if within expiration window
        if (existingOtp.getOtpExpiration() > currentTime) {
            long timeDifference = existingOtp.getOtpExpiration() - currentTime;
            String waitTime = formatWaitTime(timeDifference);
            logger.error("Rate limit exceeded. Wait time: {}", waitTime);
            throw new CustomMessagePresentException("Please wait " + waitTime + " before requesting a new OTP");
        }

        // Check if user exceeded max attempts and needs to wait longer
        if (existingOtp.getOtpCount() >= MAX_OTP_ATTEMPTS) {
            long blockEndTime = existingOtp.getOtpExpiration() + (EXPIRE_HOURS * HOURS_TO_MILLIS);
            if (blockEndTime > currentTime) {
                long timeDifference = blockEndTime - currentTime;
                String waitTime = formatWaitTime(timeDifference);
                logger.error("Max attempts exceeded. Block time: {}", waitTime);
                throw new CustomMessagePresentException("Maximum attempts exceeded. Please wait " + waitTime);
            }
            // Reset count if block period has passed
            existingOtp.setOtpCount(0);
        }
    }

    private void updateExistingOtp(OTP existingOtp, String mobile) {
        String otpCode = generateOtpCode();
        
        if (!sendSmsOtp(mobile, otpCode)) {
            throw new CustomMessagePresentException("Failed to send OTP");
        }

        long otpExpiration = System.currentTimeMillis() + (EXPIRE_MINUTES * MINUTES_TO_MILLIS);
        existingOtp.setOtpCode(otpCode);
        existingOtp.setOtpExpiration(otpExpiration);
        existingOtp.setOtpCount(existingOtp.getOtpCount() + 1);
        otpRepository.save(existingOtp);
    }

    private void createNewOtp(User user, String mobile) {
        String otpCode = generateOtpCode();
        
        if (!sendSmsOtp(mobile, otpCode)) {
            throw new CustomMessagePresentException("Failed to send OTP");
        }

        saveNewOtp(user, otpCode);
    }

    private boolean isOtpExpired(long expirationTime) {
        return expirationTime < System.currentTimeMillis();
    }

    private boolean isValidOtp(String storedOtp, String providedOtp) {
        return storedOtp != null && storedOtp.equals(providedOtp);
    }

    private void clearOtpAndVerifyUser(OTP otp, User user) {
        otp.setOtpCode(null);
        otpRepository.save(otp);

        user.setIsPhoneNumberVerified(true);
        userRepository.save(user);
    }

    private String generateOtpCode() {
        SecureRandom secureRandom = new SecureRandom();
        return String.format("%04d", secureRandom.nextInt(OTP_UPPER_BOUND));
    }

    private void saveNewOtp(User user, String otpCode) {
        OTP otp = new OTP();
        long otpExpiration = System.currentTimeMillis() + (EXPIRE_MINUTES * MINUTES_TO_MILLIS);
        otp.setUser(user);
        otp.setOtpCode(otpCode);
        otp.setOtpExpiration(otpExpiration);
        otp.setOtpCount(1);
        otpRepository.save(otp);
    }

    private boolean sendSmsOtp(String mobile, String otpCode) {
        try {
            String token = authenticateAndGetToken();
            return sendSmsWithToken(mobile, otpCode, token);
        } catch (Exception e) {
            logger.error("Failed to send SMS OTP to: {}", mobile, e);
            return false;
        }
    }

    private String authenticateAndGetToken() {
        try {
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);

            HttpEntity<String> request = new HttpEntity<>(LOGIN_REQUEST_BODY, headers);
            ResponseEntity<String> response = restTemplate.postForEntity(smsLoginApiUrl, request, String.class);

            SMSResponse loginResponse = objectMapper.readValue(response.getBody(), SMSResponse.class);
            
            if (loginResponse.getObj() == null || loginResponse.getObj().getToken() == null) {
                throw new RuntimeException("Failed to obtain authentication token");
            }
            
            return loginResponse.getObj().getToken();
        } catch (RestClientException | Exception e) {
            logger.error("Failed to authenticate with SMS service", e);
            throw new RuntimeException("SMS service authentication failed", e);
        }
    }

    private boolean sendSmsWithToken(String mobile, String otpCode, String token) {
        try {
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);
            headers.setBearerAuth(token);

            String message = String.format(OTP_MESSAGE_TEMPLATE, otpCode);
            String requestBody = String.format(SMS_REQUEST_BODY_TEMPLATE, mobile, message);

            HttpEntity<String> request = new HttpEntity<>(requestBody, headers);
            ResponseEntity<String> response = restTemplate.postForEntity(smsSendApiUrl, request, String.class);

            SMSResponse smsResponse = objectMapper.readValue(response.getBody(), SMSResponse.class);
            return smsResponse.isSuccess();
        } catch (RestClientException | Exception e) {
            logger.error("Failed to send SMS to: {}", mobile, e);
            return false;
        }
    }

    private String formatWaitTime(long timeDifferenceMillis) {
        long hours = timeDifferenceMillis / HOURS_TO_MILLIS;
        long minutes = (timeDifferenceMillis % HOURS_TO_MILLIS) / MINUTES_TO_MILLIS;
        long seconds = (timeDifferenceMillis % MINUTES_TO_MILLIS) / 1000;

        StringBuilder waitTime = new StringBuilder();
        if (hours > 0) {
            waitTime.append(hours).append(" hour").append(hours > 1 ? "s" : "");
        }
        if (minutes > 0) {
            if (waitTime.length() > 0) waitTime.append(" and ");
            waitTime.append(minutes).append(" minute").append(minutes > 1 ? "s" : "");
        }
        if (seconds > 0 || waitTime.length() == 0) {
            if (waitTime.length() > 0) waitTime.append(" and ");
            waitTime.append(seconds).append(" second").append(seconds != 1 ? "s" : "");
        }

        return waitTime.toString();
    }

    /**
     * Retrieve user by Firebase User ID or throw an exception.
     */
    private User findByFirebaseUserIdOrThrow(String firebaseUserId) {
        return userRepository.findByFirebaseUserId(firebaseUserId)
                .orElseThrow(() -> new CustomMessagePresentException("User does not exist for this Firebase User Id"));
    }
}
```
