### 混合排序

~~~sql
SELECT *
FROM my_order o
		 INNER JOIN my_appraise a ON a.orderid = o.id
ORDER BY a.is_reply ASC,
		     a.appraise_time DESC
LIMIT 0,20
~~~

<b>优化</b>

~~~sql
SELECT *
FROM ((SELECT *
      	FROM my_order o
      		   INNER JOIN my_appraise a
      						 ON a.orderid = o.id
      							  AND is_reply = 0
       	ORDER BY appraise_time DESC
        LIMIT 0,20)
        UNION ALL
        (SELECT * 
         FROM my_order o
        			INNER JOIN my_appraise a
      						 ON a.orderid = o.id
      							  AND is_reply = 1
         ORDER BY appraise_time DESC
        LIMIT 0,20)) t
ORDER BY is_reply ASC,
				appraisetime DESC
LIMIT 20;
~~~



### EXIST语句

~~~SQL
SELECT *
FROM   my_neighbor n 
       LEFT JOIN my_neighbor_apply sra 
              ON n.id = sra.neighbor_id 
                 AND sra.user_id = 'xxx' 
WHERE  n.topic_status < 4 
       AND EXISTS(SELECT 1 
                  FROM   message_info m 
                  WHERE  n.id = m.neighbor_id 
                         AND m.inuser = 'xxx') 
       AND n.topic_type <> 5 
~~~

<b>优化 </b>INNER JOIN代替EXISTS

~~~sql
SELECT *
FROM   my_neighbor n 
       INNER JOIN message_info m 
               ON n.id = m.neighbor_id 
                  AND m.inuser = 'xxx' 
       LEFT JOIN my_neighbor_apply sra 
              ON n.id = sra.neighbor_id 
                 AND sra.user_id = 'xxx' 
WHERE  n.topic_status < 4 
       AND n.topic_type <> 5 
~~~

