## `STRING_AGG` function with subquery in first param not work so alternative solution is
```SQL
WITH TaughtSubjects AS (
SELECT
	ejedts.EMPLOYEE_JOB_EXPERIENCE_DETAIL_ID,
	STRING_AGG(s.SUBJECT_NAME_BN,
	', ') AS subjects
FROM
	EMPLOYEE_JOB_EXPERIENCE_DETAIL_TAUGHT_SUBJECT AS ejedts
LEFT JOIN SUBJECT s ON
	s.SUBJECT_ID = ejedts.SUBJECT_ID
GROUP BY
	ejedts.EMPLOYEE_JOB_EXPERIENCE_DETAIL_ID
)

SELECT
	TaughtSubjects.subjects AS taughtSubjects
FROM
	EMPLOYEE_JOB_EXPERIENCE_DETAIL_LATEST ejedl
LEFT JOIN TaughtSubjects ON
	ejedl.EMPLOYEE_JOB_EXPERIENCE_DETAIL_ID = TaughtSubjects.EMPLOYEE_JOB_EXPERIENCE_DETAIL_ID
```

## If Bangla text shows ???? in output, then 'N' should concat first like
```SQL
DECLARE @IS_RECOGNIZED_UNI_INST_REQUIRED NVARCHAR(4000) = N'স্বীকৃত বিশ্ববিদ্যালয় হতে '
```
