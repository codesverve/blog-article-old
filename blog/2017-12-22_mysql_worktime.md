# mysql计算两个日期间的工作时长（剔除周末，设定上班时间） 

## SQL方法创建

```
# 查询时间段内的工作时间函数(剔除周末与非工作时间)
# 参数解释
# _st: 开始时间
# _ed: 结束时间
# _hour1: 工作时间开始小时
# _hour2: 工作时间结束小时
# _minute1: 工作时间开始分钟
# _minute2: 工作时间结束分钟
# 返回工作时间总分钟数

DELIMITER $$
CREATE FUNCTION work_minute_sum(_st DATETIME, _ed DATETIME, _hour1 INT, _hour2 INT, _minute1 INT, _minute2 INT) RETURNS INTEGER
BEGIN
    DECLARE _one_week_minute INT DEFAULT 10080;
    DECLARE _one_day_work_minute INT DEFAULT (_hour2 - _hour1) * 60 - _minute1 + _minute2;
    DECLARE _st_hour INT DEFAULT DATE_FORMAT(_st, '%H');
    DECLARE _ed_hour INT DEFAULT DATE_FORMAT(_ed, '%H');
    DECLARE _st_minute INT DEFAULT DATE_FORMAT(_st, '%i');
    DECLARE _ed_minute INT DEFAULT DATE_FORMAT(_ed, '%i');
    DECLARE _st_week INT DEFAULT DATE_FORMAT(_st, '%w');
    DECLARE _ed_week INT DEFAULT DATE_FORMAT(_ed, '%w');
    
    DECLARE _diffminute INT;
    DECLARE _che DATETIME;
    DECLARE _flag INT DEFAULT 1;
    DECLARE _week INT;
    DECLARE _hour_div INT;
    DECLARE _count_week INT DEFAULT 0;
    DECLARE _work_minute INT DEFAULT 0;
    
    IF ((_hour1 > _hour2) || (_hour1 = _hour2 && _minute1 >= _minute2)) THEN
        RETURN 0;
    END IF;
    IF _st > _ed THEN 
        SET _che = _st;
        SET _st = _ed;
        SET _ed = _che;
        SET _flag = -1;
    END IF;

    SET _diffminute = TIMESTAMPDIFF(MINUTE, _st, _ed);
    SET _count_week = _diffminute / _one_week_minute;
    SET _work_minute = _count_week * 5 * _one_day_work_minute;
    
    SET _st = DATE_ADD(_st, INTERVAL (_count_week * 7) DAY);

    IF (_st_week = 0 || _st_week = 6) THEN
        SET _work_minute = _work_minute;
    ELSEIF ((_st_hour > _hour2) || (_st_hour = _hour2 && _st_minute >= _minute2)) THEN
        SET _work_minute = _work_minute - _one_day_work_minute;
	ELSEIF (_st_hour > _hour1) || (_st_hour = _hour1 && _st_minute > _minute1) THEN
        SET _hour_div = _st_hour - _hour1;
        SET _work_minute = _work_minute - (_hour_div * 60) - _st_minute + _minute1;
    END IF;
    
    IF _ed_week = 0 || _ed_week = 6 || _ed_hour < _st_hour || (_ed_hour = _st_hour && _ed_minute <= _st_minute) THEN
        SET _work_minute = _work_minute;
    ELSEIF (_ed_hour < _hour1) || (_ed_hour = _hour1 && _ed_minute < _minute1) THEN
        SET _work_minute = _work_minute - _one_day_work_minute;
    ELSEIF (_ed_hour < _hour2) || (_ed_hour = _hour2 && _ed_minute < _minute2) THEN
		SET _hour_div = _hour2 - _ed_hour;
        SET _work_minute = _work_minute - (_hour_div * 60) + _ed_minute - _minute2;
    END IF;
    
    loop_cal:LOOP
        SET _week = DATE_FORMAT(_st, '%w');
        IF _st >= _ed THEN
            LEAVE loop_cal;
        END IF;
        IF _week != 0 && _week != 6 THEN
		    SET _work_minute = _work_minute + _one_day_work_minute;
        END IF;
        SET _st = DATE_ADD(_st, INTERVAL 1 DAY);
    END LOOP;
    
    RETURN _work_minute * _flag;
END $$
DELIMITER ;
```

## 调用方法
#例：某公司工作时间为上午： 09:00 ~ 12:00，下午： 13:30 ~ 18:00，则在2019年3月3日0点到2019年3月13日22点的总工作时间是：
select work_minute_sum('2019-03-03 00:00:00', '2019-03-13 22:00:00', 9, 12, 0, 0) + work_minute_sum('2019-03-03 00:00:00', '2019-03-13 22:00:00', 13, 18, 30, 0);
