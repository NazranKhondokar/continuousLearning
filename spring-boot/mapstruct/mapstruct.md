Multi-layered applications often require to map between different object models (e.g. entities and DTOs). Writing such mapping code is a tedious and error-prone task. MapStruct aims at simplifying this work by automating it as much as possible. Check [MapStruct Performance](https://www.baeldung.com/java-performance-mapping-frameworks#1averagetime)

## Installation
- When using a modern version of Gradle (>= 4.6), you add something along the following lines to your build.gradle:

```java
dependencies {
    ...
    implementation 'org.mapstruct:mapstruct:1.5.5.Final'
 
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.5.Final'
}
```

## Make a target model

```java
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
public class EmployeeJobInfoResponse {
    private String designationName;
    private String subjectName;
    private String taughtSubjects;
}
```

## Make a source model
```java
public interface JobExperienceProjection {
    String getDesignationName();
    String getSubjectName();
    String getTaughtSubjects();
}
```

## Make a Mapper
```java
...
import org.mapstruct.Mapper;
import org.mapstruct.factory.Mappers;

@Mapper(componentModel = "spring")
public interface JobInfoMapper {

    JobInfoMapper MAPPER = Mappers.getMapper(JobInfoMapper.class);

    EmployeeJobInfoResponse mapToEmployeeJobInfoResponse(JobExperienceProjection experienceProjection);
}
```

## Call the mapper from service or other
- Here `jobRepository.findJob(id)` returns `JobExperienceProjection` after executing some query

```java
...
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Slf4j
@Service
@RequiredArgsConstructor
public class EmployeeProfileServiceImpl implements EmployeeProfileService {

    private final EmployeeJobExperienceDetailRepository jobRepository;

    @Override
    public EmployeeJobInfoResponse findJobInfoById(Long id) {

        return JobInfoMapper.MAPPER.mapToEmployeeJobInfoResponse(jobRepository.findJobById(id));
    }
}
```

## If we need multiple sources but a single target then 
- Create another source model

```java
public interface EmploymentProjection {
    Integer getRecommendedMeritPosition();
    String getBatch();
    String getRecruitedDesignationName();
}
```
## Extend target model

```java
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Data
@NoArgsConstructor
public class EmployeeJobInfoResponse {

    private String designationName;
    private String subjectName;
    private String taughtSubjects;
    // extended attributes
    private String batch;
    private Integer recommendedMeritPosition;
    private String recruitedDesignationName;
}
```

## Pass multi-source at Mapper

```java
...
    EmployeeJobInfoResponse mapToEmployeeJobInfoResponse(JobExperienceProjection experienceProjection,
                                                            EmploymentProjection employmentProjection);
...
```

## Call it from service

```java
...
    private final EmployeeJobExperienceDetailRepository jobRepository;
    private final EmployeeEmploymentRepository employmentRepository;

    @Override
    public EmployeeJobInfoResponse findJobInfoById(Long id) {

        return JobInfoMapper.MAPPER.mapToEmployeeJobInfoResponse(jobRepository.findJobById(id),
                                                                employmentRepository.findEmploymentById(id));
    }
...
```
