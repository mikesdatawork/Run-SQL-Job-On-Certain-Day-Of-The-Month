> **Note** — The folder `linguist-samples/` contains tiny real files so GitHub can correctly display all languages used in this repo.  
> The actual content and examples remain in this README.

![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Run SQL Job On Certain Day Of The Month
**Post Date: September 23, 2016**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>Someone recently asked me how to setup a job to run on a certain day. In this case it's the 3rd day of the month. There is one simple solution which can do that. Unfortunately; it means a job has to run every day, but that's not a big deal really.
You just have to create Step 1 to check on today. If today is NOT the 3rd day of the month then STOP the current job. If it is the current day, then do nothing and the job will proceed to the next step.</p> 



## SQL-Logic
```SQL
use master;
set nocount on
 
declare @today datetime = (select dateadd(d, -0, datediff(d, 0, getdate())))
declare @3rd   datetime = (select dateadd(day, 2, dateadd(month, datediff(month, 0, getdate()), 0)))
 
if @today <> @3rd
    begin
        print 'today is NOT the 3rd day'
        exec msdb..sp_stop_job 'my job name'
    end
        else
            print 'today is the 3rd'
```

![Declare Days With SQL]( https://mikesdatawork.files.wordpress.com/2016/09/image0013.png "declare days for sql")
 
You might be thinking… "Doesn't this throw an error, or alert on the job?" Nope :) Basically what happens is the Job does not 'fail'. It's simply 'cancelled'. This is basically the same thing whenever you right-click a job and select 'Stop Job'. So although the agent history will show a nasty red icon next to the run history; it's not a 'failure'. It still only means it was stopped (cancelled) as you can see below by the Job history.
If you're interested; here's the quick test I created.
I created a 2 step Job.
Step 1: Check to see if it's the 3rd day. If not; Stops the Job. If so… do nothing and continue to the next step as usual.
Step 2: Backup the Model database.
Here's the results of the Job after it's cancelled from within the Step 1.

![Run Job Certain Day]( https://mikesdatawork.files.wordpress.com/2016/09/image003.png "Run Job Certain Day")
 
I ran the Job a second time where @today = @today just to confirm the success, and it worked no problem.
Now; if you're looking for a business day; you can try this:



## SQL-Logic
```SQL
use master;
set nocount on
 
declare @year_start        datetime
declare @year_finish       datetime
declare @today             datetime = (select dateadd(d, -0, datediff(d, 0, getdate())))
declare @daydiff           int
declare @holidays          table (Holiday datetime)
declare @3rd_biz_day       table
       (
              [Date]               datetime
       ,      [MonthName]          varchar (50)
       ,      [DateName]           varchar (50)
       ,      [BusinessDay] int
       )
set           @year_start   ='01 jan 2016'
set           @year_finish  ='31 dec 2016'
set           @daydiff      = datediff (day,@year_start,@year_finish)+1
 
insert into @holidays                    -- US Holidays
       select '1/1/2016'    union all     -- New Year's Day
       select '1/18/2016'   union all     -- Martin Luther King, Jr. Day
       select '2/15/2016'   union all     -- George Washington's Birthday
       select '3/30/2016'   union all     -- Memorial Day
       select '6/4/2016'    union all     -- Independence Day
       select '9/5/2016'    union all     -- Labor Day
       select '10/10/2016'  union all     -- Columbus Day
       select '11/11/2016'  union all     -- Veterans Day
       select '11/24/2016'  union all     -- Thanksgiving Day
       select '12/26/2016'               -- Christmas Day
 
insert into @3rd_biz_day
select * from 
       (
              select 
                     'Date'        = dateadd(day,daynum,@year_start)
              ,      'MonthName'   = datename(month,(dateadd(day,daynum,@year_start)))
              ,      'DateName'    = datename(weekday,(dateadd(day,daynum,@year_start)))
              ,      'BusinessDay' = 
                     row_number() over (partition by (datepart(month,(dateadd(day,daynum,@year_start)))) 
                     order by (dateadd(day,daynum,@year_start)))
                     from (select top (@daydiff) row_number() over(order by (select 1))-1 as daynum 
              from 
                     sys.syscolumns sc_a cross join sys.syscolumns sc_b) dates
              where
                     datename(weekday,(dateadd(day,daynum,@year_start))) not in ('saturday','sunday') 
                     and dateadd(day,daynum,@year_start) not in (select holiday from @holidays)
       )      sc_a
where BusinessDay = 3
 
if     @today not in(select [date] from @3rd_biz_day)
       begin
              print 'today is NOT the 3rd business day'
              exec  msdb..sp_stop_job 'my job name'
       end
       else
              print 'today is the 3rd business day - continue to next job step'
```


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)


## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")
