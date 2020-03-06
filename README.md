# utl-transposing-and-summarizing-by-group-using-sql-arrays
Transposing and summarizing by group using sql arrays
    Transposing and summarizing by group using sql arrays

    github
    https://tinyurl.com/vrqqyc3
    https://github.com/rogerjdeangelis/utl-transposing-and-summarizing-by-group-using-sql-arrays

    The op wanted a sql solution.

    The transpose macro or proc transpose are more appropriate?

    Not sure I would do this.
    This is more of an achademic exercise.

    This may not be as slow as you think. It depends on hpw teradata or oracle partition the problem across cores?

    SAS Forum
    https://tinyurl.com/yxxqmgl8
    https://communities.sas.com/t5/SAS-Enterprise-Guide/proc-sql-for-transpose/m-p/627170/highlight/true#M35582

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    data have;
     infile cards4;
     input Name$ Visit;
    cards4;
    A 1
    A 2
    A 3
    B 1
    B 3
    B 7
    C 2
    D 8
    ;;;;
    run;quit;


    WORK.HAVE total obs=8

      NAME    VISIT

       A        1
       A        2
       A        3
       B        1
       B        3
       B        7
       C        2
       D        8

    *               _   _               _
     _ __ ___   ___| |_| |__   ___   __| |
    | '_ ` _ \ / _ \ __| '_ \ / _ \ / _` |
    | | | | | |  __/ |_| | | | (_) | (_| |
    |_| |_| |_|\___|\__|_| |_|\___/ \__,_|

    ;

    * create a template with all possible visits
    enen the ones not on have;


    WORK.HAVTPL total obs=1

      V1    V2    V3    V4    V5    V6    V7    V8

       1     2     3     4     5     6     7     8

    Cartesion join the the template with the data

    NAME VISIT  V1    V2    V3    V4    V5    V6    V7    V8

     A     1     1     2     3     4     5     6     7     8
     A     2     1     2     3     4     5     6     7     8
     A     3     1     2     3     4     5     6     7     8
     B     1     1     2     3     4     5     6     7     8
     B     3     1     2     3     4     5     6     7     8
     B     7     1     2     3     4     5     6     7     8
     C     2     1     2     3     4     5     6     7     8
     D     8     1     2     3     4     5     6     7     8

    Substitute 1 where the visit matches one of the v1-v8 else substiyute 0

    Finally sum the V variables by name

    WORK.HAVJYN total obs=4

    sets V1-V3 to 1 because of the first three rows of have

              ==============
      NAME    V1    V2    V3    V4    V5    V6    V7    V8

       A       1     1     1     0     0     0     0     0
    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;

    WORK.HAVJYN total obs=4

      NAME    V1    V2    V3    V4    V5    V6    V7    V8

       A       1     1     1     0     0     0     0     0
       B       1     0     1     0     0     0     1     0
       C       0     1     0     0     0     0     0     0
       D       0     0     0     0     0     0     0     1

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    ;


    %arraydelete(vs); * delete macro array if it exists;

    %put &=vs1;

    %array(vs,VALUES=1-8); * create macro array vs1=1, vs2=2 ... vsn-8;

    proc sql;
       create
          table havTpl as
        select
          %do_over(vs,phrase=? as V? length=3,between=comma)
        from
          have(obs=1)
    ;
       create
         table want as
           select
              name
             ,%do_over(vs,phrase=sum(v?) as v?,between=comma)
           from
             (
              select
                 l.*
                ,%do_over(vs,phrase=case when (v?=visit) then 1 else 0 end as v?,between=comma)
              from
                 have as l, havTpl as r
              )
       group
          by name
    ;quit;

    You can get the generated code;

    *              _                                        _           _
      ___ ___   __| | ___    __ _  ___ _ __   ___ _ __ __ _| |_ ___  __| |
     / __/ _ \ / _` |/ _ \  / _` |/ _ \ '_ \ / _ \ '__/ _` | __/ _ \/ _` |
    | (_| (_) | (_| |  __/ | (_| |  __/ | | |  __/ | | (_| | ||  __/ (_| |
     \___\___/ \__,_|\___|  \__, |\___|_| |_|\___|_|  \__,_|\__\___|\__,_|
                            |___/
    ;

    data gencode;

     put %do_over(vs,phrase="? as V? length=3" /);
     put %do_over(vs,phrase="sum(v?) as v?" /);
     put %do_over(vs,phrase="case when (v?=var) then 1 else 0 end as v?" /);

    run;quit;


    1 as V1 length=3
    2 as V2 length=3
    3 as V3 length=3
    4 as V4 length=3
    5 as V5 length=3
    6 as V6 length=3
    7 as V7 length=3
    8 as V8 length=3

    sum(v1) as v1
    sum(v2) as v2
    sum(v3) as v3
    sum(v4) as v4
    sum(v5) as v5
    sum(v6) as v6
    sum(v7) as v7
    sum(v8) as v8

    case when (v1=var) then 1 else 0 end as v1
    case when (v2=var) then 1 else 0 end as v2
    case when (v3=var) then 1 else 0 end as v3
    case when (v4=var) then 1 else 0 end as v4
    case when (v5=var) then 1 else 0 end as v5
    case when (v6=var) then 1 else 0 end as v6
    case when (v7=var) then 1 else 0 end as v7
    case when (v8=var) then 1 else 0 end as v8

    * __       _
     / _|_   _(_)
    | |_| | | | |
    |  _| |_| | |
    |_|  \__, |_|
         |___/
    ;

    I updated the arraydelete macro in

    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories

    Added nowarn and sydel for XXn count

    %macro arraydelete(pfx)/des="Delete array macrovariables create by array macro";
      %do i= 1 %to &&&pfx.n;
          %symdel &pfx&i / nowarn;
      %end;
      %symdel  &&pfx.n / nowarn;
    %mend arraydelete;


