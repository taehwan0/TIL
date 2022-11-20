# WebGoat

## INDEX
- [Injection](#injection)

## Injection

### Intro

#### What is SQL?

SQL은 관계형 데이터베이스를 관리하고, 그 안의 데이터에 대해 다양한 작업을 할 수 있도록 하는 표준화된 언어이다.
데이터베이스는 데이터의 모음으로 행, 열 및 테이블로 구성되며 정보를 


**PROBLEM**
> Look at the example table.
> Try to retrieve the department of the employee Bob Franco.
> Note that you have been granted full administrator privileges in this assignment and can access all data without authentication.


**SOLUTION**
```SQL
SELECT * 
FROM employees 
WHERE first_name like 'Bob'
  AND last_name = 'Franco'
```

`'`, `"` 로 끊고 1=1 등의 `TRUE` 를 만드는 방법,
`--` 를 넣어서 주석을 만드는 방법
