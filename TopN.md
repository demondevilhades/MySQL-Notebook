# TopN
## Get the top N of each grous in MySQL 8

### 0. Prepare
```SQL
-- score definition

CREATE TABLE `score` (
  `id` int NOT NULL AUTO_INCREMENT,
  `subject` varchar(32) COLLATE utf8mb4_unicode_ci NOT NULL,
  `result` int DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

INSERT INTO score (`subject`, `result`) VALUES('A', 98);
INSERT INTO score (`subject`, `result`) VALUES('A', 96);
INSERT INTO score (`subject`, `result`) VALUES('A', 94);
INSERT INTO score (`subject`, `result`) VALUES('A', 92);
INSERT INTO score (`subject`, `result`) VALUES('A', 90);
INSERT INTO score (`subject`, `result`) VALUES('A', 88);
INSERT INTO score (`subject`, `result`) VALUES('A', 86);
INSERT INTO score (`subject`, `result`) VALUES('A', 85);
INSERT INTO score (`subject`, `result`) VALUES('A', 87);
INSERT INTO score (`subject`, `result`) VALUES('A', 89);
INSERT INTO score (`subject`, `result`) VALUES('A', 91);
INSERT INTO score (`subject`, `result`) VALUES('A', 93);
INSERT INTO score (`subject`, `result`) VALUES('A', 95);
INSERT INTO score (`subject`, `result`) VALUES('A', 97);
INSERT INTO score (`subject`, `result`) VALUES('A', 99);
INSERT INTO score (`subject`, `result`) VALUES('B', 69);
INSERT INTO score (`subject`, `result`) VALUES('B', 69);
INSERT INTO score (`subject`, `result`) VALUES('B', 68);
INSERT INTO score (`subject`, `result`) VALUES('B', 68);
INSERT INTO score (`subject`, `result`) VALUES('B', 67);
INSERT INTO score (`subject`, `result`) VALUES('B', 67);
INSERT INTO score (`subject`, `result`) VALUES('B', 66);
INSERT INTO score (`subject`, `result`) VALUES('B', 66);
INSERT INTO score (`subject`, `result`) VALUES('B', 65);
INSERT INTO score (`subject`, `result`) VALUES('C', 79);
INSERT INTO score (`subject`, `result`) VALUES('C', 78);
INSERT INTO score (`subject`, `result`) VALUES('C', 78);
INSERT INTO score (`subject`, `result`) VALUES('C', 77);
INSERT INTO score (`subject`, `result`) VALUES('C', 77);
INSERT INTO score (`subject`, `result`) VALUES('C', 76);
INSERT INTO score (`subject`, `result`) VALUES('C', 76);
INSERT INTO score (`subject`, `result`) VALUES('C', 75);
INSERT INTO score (`subject`, `result`) VALUES('C', 75);
INSERT INTO score (`subject`, `result`) VALUES('C', 74);
INSERT INTO score (`subject`, `result`) VALUES('C', 74);
INSERT INTO score (`subject`, `result`) VALUES('C', 73);
```

### 2. Go
```SQL
-- Retrieve the top 3 in each class and subject, innoring the same results. Should be 18 (3+2+3) records. 
SELECT 
    s.`id`, s.`subject`, s.`result` 
FROM `score` s 
WHERE (SELECT COUNT(1) FROM `score` s0 WHERE s.`subject` = s0.`subject` AND s.`result` <= s0.`result`) <= 3 
ORDER BY s.`subject`, s.`result` DESC
;

-- Retrieve the top 3 in each class and subject, without innoring the same results. Should be 18 (3+2+3) records. 
SELECT 
    s.`id`, s.`subject`, s.`result` 
FROM `score` s 
WHERE (SELECT COUNT(1) FROM `score` s0 WHERE s.`subject` = s0.`subject` AND s.`result` < s0.`result`) < 3 
ORDER BY s.`subject`, s.`result` DESC
;


-- Retrieve the top 2 in each class and subject, without innoring the same results. Add ranking. Should be 13 records. 
SET @cur_rank := NULL;
SET @cur_subject := NULL;
SET @prev_result := NULL;

SELECT 
    (CASE 
        WHEN (@cur_subject = s.`subject`) AND (@prev_result = s.`result`) THEN NULL 
        WHEN (@cur_subject = s.`subject`) THEN (@cur_rank := @cur_rank + 1) 
        ELSE (@cur_rank := 1) AND (@cur_subject := s.`subject`) 
    END) AND (@prev_result := s.`result`) AND 0 AS `temp`, 
    s.`id`, s.`subject`, s.`result`, 
    @cur_rank AS `rank` 
FROM `score` s 
WHERE (SELECT COUNT(1) FROM `score` s0 WHERE s.`subject` = s0.`subject` AND s.`result` < s0.`result`) < 3 
ORDER BY s.`subject`, s.`result` DESC
;

-- Using DENSE_RANK()
SELECT 
    s.`id`, s.`subject`, s.`result`, 
    DENSE_RANK() OVER ( 
      PARTITION BY s.`subject` 
      ORDER BY s.`result` DESC 
    ) `rank`
FROM `score` s 
ORDER BY s.`subject`, `rank` ASC
;


SELECT 
    s0.`id`, s0.`subject`, s0.`result`, s0.`rank`
FROM ( 
    SELECT 
        s.`id`, s.`subject`, s.`result`, 
        DENSE_RANK() OVER ( 
          PARTITION BY s.`subject` 
          ORDER BY s.`result` DESC 
        ) `rank`
    FROM `score` s ) s0 
WHERE `rank` <= 3 
ORDER BY `subject`, `result` DESC
;

```
