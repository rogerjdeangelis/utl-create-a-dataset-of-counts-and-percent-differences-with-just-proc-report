# utl-create-a-dataset-of-counts-and-percent-differences-with-just-proc-report
Create a dataset of counts and percent differences with just proc report.

    Create a dataset of counts and percent differences with just proc report

    Report is more flexible that tabulate, transpose, freq, sort, summarize, print and even 'proc corr.
    I think report is underused for creating datasets.

    github
    https://tinyurl.com/y88yf7c8
    https://github.com/rogerjdeangelis/utl-create-a-dataset-of-counts-and-percent-differences-with-just-proc-report

    SAS Forum
    https://tinyurl.com/y7w34nz4
    https://communities.sas.com/t5/New-SAS-User/Calculating-Percentages-in-Total-Row/m-p/516458

    Related report repos
    https://tinyurl.com/y7dygfdq
    https://github.com/rogerjdeangelis?utf8=%E2%9C%93&tab=repositories&q=report+in%3Aname&type=&language=


    INPUT
    =====

     WORK.HAVE total obs=18

                      |  RULES     COUNTS    % Diff
                                   ------
       ZON     VALUE  |  VALUE  ZONE1  ZONE2  PCTZ2_Z1
                      |
      ZONE1      A    |  A        2      1     -50.0  (100*(2-1)/2 = -50
      ZONE1      A    |  B        2      3      50.0
      ZONE1      B    |  C        1      2     100.0
      ZONE1      B    |
      ZONE1      C    |

      ZONE2      B    |
      ZONE2      B    |
      ZONE2      B    |
      ZONE2      C    |
      ZONE2      A    |
      ZONE2      C    |

      ZONE3      C    |
      ZONE3      C    |
      ZONE3      C    |
      ZONE3      B    |
      ZONE3      B    |
      ZONE3      A    |
      ZONE3      A    |


    EXAMPLE OUTPUT
    --------------

     WANT total obs=4

       VALUE    ZONE1    ZONE2    ZONE3    PCTZ2_Z1    PCTZ3_Z2

       A          2        1        2       -50.0       100.0
       B          2        3        2        50.0       -33.3
       C          1        2        3       100.0        50.0

       TOTAL      5        6        7        20.0        16.7


    PROCESS
    =======

     proc report data=have nowd out=want (drop=_break_ rename=(%utl_renamel(
        _c2_ _c3_ _c4_ pct1 pct2, zone1 zone2 zone3 pctZ2_Z1 pctZ3_Z2)));

       attrib pctZ2_Z1 : format=5.1 pctZ3_Z2 : format=5.1;

       column VALUE ZON pct1 pct2;

       define VALUE /group '';
       define ZON /across '';
       define pct1 /computed  format=5.1;
       define pct2 /computed  format=5.1;

       compute pct1;
             pct1=100*(_C3_-_C2_)/_C2_;
       endcomp;

       compute pct2;
             pct2=100*(_C4_-_C3_)/_C3_;
       endcomp;

       rbreak after /summarize;
       compute after;
             VALUE='TOTAL';
       endcomp;
     run;quit;

    OUTPUT
    ======
    see above


    *
     _ __ ___   __ _  ___ _ __ ___
    | '_ ` _ \ / _` |/ __| '__/ _ \
    | | | | | | (_| | (__| | | (_) |
    |_| |_| |_|\__,_|\___|_|  \___/

    ;

    %macro utl_renamel ( old , new ) ;
        /* Take two cordinated lists &old and &new and  */
        /* return another list of corresponding pairs   */
        /* separated by equal sign for use in a rename  */
        /* statement or data set option.                */
        /*                                              */
        /*  usage:                                      */
        /*    rename = (%renamel(old=A B C, new=X Y Z)) */
        /*    rename %renamel(old=A B C, new=X Y Z);    */
        /*                                              */
        /* Ref: Ian Whitlock <whitloi1@westat.com>      */

        %local i u v warn ;
        %let warn = Warning: RENAMEL old and new lists ;
        %let i = 1 ;
        %let u = %scan ( &old , &i ) ;
        %let v = %scan ( &new , &i ) ;
        %do %while ( %quote(&u)^=%str() and %quote(&v)^=%str() ) ;
            &u = &v
            %let i = %eval ( &i + 1 ) ;
            %let u = %scan ( &old , &i ) ;
            %let v = %scan ( &new , &i ) ;
        %end ;

        %if (null&u ^= null&v) %then
            %put &warn do not have same number of elements. ;

    %mend  utl_renamel ;


