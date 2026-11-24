# jnr-Jenner-analytics-dynamic-code-generation-pivot-wide-sql-arrays-do-over
Jenner Analytics dynamic code generation with enhanced array and do over macro
    /*---
    c:/utl/jnr-Jenner-analytics-dynamic-code-generation-pivot-wide-sql-arrays-do-over.sas
    ---*/

    Jenner Analytics dynamic code generation with enhanced array and do over macro

    Pivot student semester grades from a long table to wide table

    Obs TERM STUDENT SCORE   STUDENT SCORE1 SCORE2 SCORE3
     1    1     A      77      A         77     89     71
     2    1     B      70      B         70     83     94
     3    1     C      80      C         80     73      .
     4    2     A      89
     5    2     B      83
     6    2     C      73
     7    3     A      71
     8    3     B      94

    CONTENTS
     1 INPUT
     2 PROCESS
     3 OUTPUT
     4 DO_OVER MACRO
     5 LOG

    /*--- 1 INPUT ---*/
    data long;
      input term student$ score @@;
    cards4;
    1 A 77 1 B 70 1 C 80
    2 A 89 2 B 83 2 C 73
    3 A 71 3 B 94
    ;;;;
    run;

    proc print;run;

    /*---
     Obs TERM STUDENT SCORE
       1   1     A      77
       2   1     B      70
       ...
       8   3     B      94
    ---*/

    /*--- 2 PROCESS ---*/
    proc sql;
     select
       distinct(term)
     into
       :_term1- from long;
    %let _termn=&sqlobs;
     select
        student
       ,%jnr_do_over(arrayName=_term,code=
        max(case when term=? then
          score else NULL end) as score?)
     from
         long
     group
          by student
    ;quit;

    /*--- 3 OUTPUT
    STUDENT score1 score2 score3
    ----------------------------
    A           77     89     71
    B           70     83     94
    C           80     73      .
    ---*/

    /*--- 4 MACRO SAVE IN YOUR AUTOCALL LIBRARY ---*/
    filename ft15f001 "c:/otojnr/jnr_do_over.sas";
    parmcards4;
    %macro jnr_do_over(
        arrayName=nams vals
       ,code=%str(x="?";y="#";output;)
       ,between=','
       )/des="Dynamic code iterating over macro code fragments";

       %if %sysfunc(countw(&arrayName))=2 %then %do;
          %let arraynamea=%scan(&arrayname,1);
          %let arraynameb=%scan(&arrayname,2);
       %end;
       %else %do;
          %let arraynamea=%scan(&arrayname,1);
          %let arraynameb=%scan(&arrayname,1);
       %end;

       %let rc=%sysfunc(dosubl(
         data work._null_ %str(;)
            length codex $32756 %str(;)
            codex='' %str(;)
            do i=1 to &&&arraynamea.n %str(;)
              code=tranwrd("&code","?",symget(cats("&arraynamea",i))) %str(;)
              code=tranwrd(code,"#",symget(cats("&arraynameb",i))) %str(;)
              codex=cats(codex,code) %str(;)
              if i ne &&&arraynamea.n then codex=cats(codex,&between) %str(;)
         end %str(;)
         call symputx("codex",codex,"G") %str(;)
       run %str(;)
       ));

       &codex

    %mend jnr_do_over;
    ;;;;
    run;

    /*--- 5 LOG ---*/

    NOTE: Copyright (c) 2026 Jenner Analytics Ltd., London, England.
    NOTE: Jenner v1.5.125 (build v1.5.125+22203d2bc1.20261120T060416Z)
          Licensed to Roger DeAngelis, Serial 3BF7B3E6.
    NOTE: Option VALIDVARNAME changed to UPCASE.
    NOTE: Option FORMCHAR changed to |----|+|---+=|-/\<>*.
    NOTE: DATA _null_

    autoexec started  24NOV2026:10:33:02

    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: Library WORKX assigned path=d:/wpswrkx.
    NOTE: Library LIBRARY assigned path=d:/wpswrkx.
    NOTE: DATA WORKX.CLASS

    NOTE: Processing inline DATALINES (19 lines)

    NOTE: The data set WORKX.CLASS has 19 observations and 5 variables.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.02 seconds
    NOTE: DATA _null_

    LOG:  24NOV2026:10:33:02
    NOTE: DATA _null_ completed. Output written to FILE PRINT
    NOTE: Option SASAUTOS changed to c:/otojnr.
    NOTE: DATA _null_

    NOTE: Reading from fileref c:/jnr/runsas_selection.sas (c:/jnr/runsas_selection.sas)
    1 data long;
    2 input term student$ score @@;
    3 cards4;
    4 1 A 77 1 B 70 1 C 80
    5 2 A 89 2 B 83 2 C 73
    6 3 A 71 3 B 94
    7 ;;;;
    8 run;
    9
    10 proc print;run;
    11
    12 /*---
    13 Obs TERM STUDENT SCORE
    14 1   1     A      77
    15 2   1     B      70
    16 ...
    17 8   3     B      94
    18 ---*/
    19
    20 proc sql;
    21 select
    22 distinct(term)
    23 into
    24 :_term1- from long;
    25 %let _termn=&sqlobs;
    26 select
    27 student
    28 ,%jnr_do_over(arrayName=_term,code=
    29 max(case when term=? then
    30 score else NULL end) as score?)
    31 from
    32 long
    33 group
    34 by student
    35 ;quit;
    36
    37 /*---
    38 STUDENT score1 score2 score3
    39 ----------------------------
    40 A           77     89     71
    41 B           70     83     94
    42 C           80     73      .
    43 ---*/

    NOTE: 43 records were read from the infile 'c:/jnr/runsas_selection.sas'.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    NOTE: DATA _null_

    autoexec completed  24NOV2026:10:33:02

    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    1          data long;
    2            input term student$ score @@;
    3          cards4;
    7          ;;;;
    8          run;
    NOTE: DATA WORK.LONG

    NOTE: Processing inline DATALINES (3 lines)

    NOTE: The data set WORK.LONG has 8 observations and 3 variables.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.00 seconds
    9
    10         proc print;run;
    NOTE: PROC PRINT data=WORK.LONG

    NOTE: PROC PRINT completed: 8 observations printed, 3 variables
    11
    12         /*---
    13          Obs TERM STUDENT SCORE
    14            1   1     A      77
    15            2   1     B      70
    16            ...
    17            8   3     B      94
    18         ---*/
    19
    20         proc sql;
    21          select
    22            distinct(term)
    23          into
    24            :_term1- from long;
    25         %let _termn=&sqlobs;
    26          select
    27             student
    28            ,%jnr_do_over(arrayName=_term,code=
    29             max(case when term=? then
    30               score else NULL end) as score?)
    31          from
    32              long
    33          group
    34               by student
    35         ;quit;
    NOTE: PROC SQL

    NOTE: Macro variable _TERM1 = 1
    NOTE: Macro variable _TERM2 = 2
    NOTE: Macro variable _TERM3 = 3
    NOTE: PROC SQL statement used.
    NOTE: DATA WORK._NULL_


    NOTE: The data set WORK._NULL_ has 1 observations and 3 variables.
    NOTE: DATA elapsed:
      wall  0.00 seconds
      cpu   0.02 seconds
    NOTE: PROC SQL

    NOTE: PROC SQL statement used.
    36
    37         /*---
    38         STUDENT score1 score2 score3
    39         ----------------------------
    40         A           77     89     71
    41         B           70     83     94
    42         C           80     73      .
    43         ---*/

    /*--- END ---*/




    
