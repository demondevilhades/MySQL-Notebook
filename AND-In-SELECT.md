# AND-In-SELECT
## AND, 0, 1, NULL

```SQL
SET @test := NULL;
SELECT (@test := 0) AND NULL, @test FROM DUAL; -- 0, 0

SET @test := NULL;
SELECT (@test := 1) AND NULL, @test FROM DUAL; -- NULL, 1

SET @test := NULL;
SELECT (@test := 1) AND 0, @test FROM DUAL; -- 0, 1


SET @test := NULL;
SELECT NULL AND (@test := 0), @test FROM DUAL; -- 0, 0

SET @test := NULL;
SELECT NULL AND (@test := 1), @test FROM DUAL; -- NULL, 1

SET @test := NULL;
SELECT 0 AND (@test := 1), @test FROM DUAL; -- 0, NULL
```
