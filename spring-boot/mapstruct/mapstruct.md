MapStruct is a tool that automates the process of creating mappings between Java bean types

## Installation
- When using a modern version of Gradle (>= 4.6), you add something along the following lines to your build.gradle:
```
dependencies {
    ...
    implementation 'org.mapstruct:mapstruct:1.5.5.Final'
 
    annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.5.Final'
}
```

## Make a target model
```
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
```
public interface JobExperienceProjection {
    String getDesignationName();
    String getSubjectName();
    String getTaughtSubjects();
}
```

## Make a Mapper
```
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
- Here `jobRepository.findJob(id)` return `JobExperienceProjection` after executing some query
```
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
    public EmployeeJobInfoResponse findJobInfoByNID(Long id) {

        return JobInfoMapper.MAPPER.mapToEmployeeJobInfoResponse(jobRepository.findJob(id));
    }
}
```

