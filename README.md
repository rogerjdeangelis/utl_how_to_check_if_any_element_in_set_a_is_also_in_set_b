# utl_how_to_check_if_any_element_in_set_a_is_also_in_set_b
How to check if any element in set A is also in set B
    How to check if any element in set A is also in set B (R, IML and SAS)

    Original post: Proc IML: If value in vector /matrix

    Four solutions

    github
    https://github.com/rogerjdeangelis/utl_how_to_check_if_set_a_is_subset_of_set_b

    Venn type problems are best solved with R or IML?
    Think outside the datastep.

    INPUT
    =====

      SD1.HAV1ST total obs=4
      Obs    A

       1     1
       2     2
       3     3
       4     4


      SD1.HAV2nd total obs=4
      Obs    B

       1     1
       2     2
       3     3
       4     4
       5     5


    WORKING CODE
    ============

        R
          any(A %in% B);

        IML (IML Rick Wicklin)

          proc iml;
            A = 6:10;
            B = 1:5;
            anyCommon = any( element(B, A) );
            print anyCommon;
          quit;

        SQL

          select a as m from sd1.hav1st
          intersect
          select b as m from sd1.hav2nd

          %put %sysfunc(ifc(&sqlobs=0,TRUE,FALSE));

        Related methods

          Hash -- Keintz, Mark
          Enhacement of Sql Søren Lassen


    OUTPUT
    ======
          R
          [1] TRUE

          SAS
          All of A is in B   TRUE



    https://goo.gl/WVCMsJ
    https://communities.sas.com/t5/SAS-IML-Software-and-Matrix/Proc-IML-If-value-in-vector-matrix/m-p/416743

    see
    https://stackoverflow.com/questions/37656853/how-to-check-if-set-a-is-subset-of-set-b-in-r

    Akun profile
    https://stackoverflow.com/users/3732271/akrun

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.hav1st(keep=a) sd1.hav2nd(keep=b);
      do a= 1,2,3,4;
        output sd1.hav1st;
      end;
      do b= 2, 5, 3, 4, 1;
        output sd1.hav2nd;
      end;
    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __  ___
    / __|/ _ \| | | | | __| |/ _ \| '_ \/ __|
    \__ \ (_) | | |_| | |_| | (_) | | | \__ \
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|___/

    ;

    *____
    |  _ \
    | |_) |
    |  _ <
    |_| \_\

    ;
    %utl_submit_r64('
    library(haven);
    library(SASxport);
    library(Hmisc);
    a<-t(read_sas("d:/sd1/hav1st.sas7bdat"));
    b<-t(read_sas("d:/sd1/hav2nd.sas7bdat"));
    a;
    b;
    any(a %in% b);
    any(b %in% a);
    ');

    *                          _
     ___  __ _ ___   ___  __ _| |
    / __|/ _` / __| / __|/ _` | |
    \__ \ (_| \__ \ \__ \ (_| | |
    |___/\__,_|___/ |___/\__, |_|
                            |_|
    ;

    proc sql;

     select
       a as mat
     from
       sd1.hav1st

     intersect

     select
       b as mat
     from
       sd1.hav2nd
    ;quit;

    %put All of A is in B   %sysfunc(ifc(&sqlobs=0,TRUE,FALSE));

    Improvement
    Søren Lassen <000002b7c5cf1459-dmarc-request@listserv.uga.edu>

    The SQL solution can easily be adopted to stop after the first observation found, just use outobs=1:
    proc sql outobs=1 noprint;
      select * from b except select * from a;
    quit;
    %put &sqlobs;


    *_               _
    | |__   __ _ ___| |__
    | '_ \ / _` / __| '_ \
    | | | | (_| \__ \ | | |
    |_| |_|\__,_|___/_| |_|

    ;

    * related solution;

    Keintz, Mark

    to me, SAS-L
    If the goal is really no more than to determine whether A is a subset of B,
    there is no need to continue checking once a single A element
    is found that is not in B, as the proc sql approach does.

    In such cases, when there is some probability of an A record not in B, this may be more efficient for large data sets:

    data a   b;
       set sashelp.class end=end_of_class;
       output b;
       if mod(_n_,2)=0 then output a;
       if _n_=1 then do; name='XXX'; output a;end;
    run;

    data _null_;
      set a end=end_of_a;
      if _n_=1 then do;
        declare hash b (dataset:'b');
          b.definekey(all:'Y');
          b.definedone();
      end;
      rc=b.find();
      if rc=0 and end_of_a=0 then return;
      text= ifc(rc=0,'A is a subset of B','A is NOT a subset of B');
      put text;
      stop;
    run;

    *___ __  __ _
    |_ _|  \/  | |
     | || |\/| | |
     | || |  | | |___
    |___|_|  |_|_____|

    ;

    Rick Wicklin via listserv.uga.edu
    Nov 20 (8 days ago)

     to SAS-L
    The SAS/IML language has several functions for dealing with sets, including union,
    intersection, set difference, and subset operations. See the article "Testing for equality of sets" at
    https://blogs.sas.com/content/iml/2012/09/06/testing-for-equality-of-sets.html
    'for an overview and to test whether A = B as sets.

    Subset testing is a "one liner": you use the ALL and ELEMENT functions to check whether all elements of A are contains in B:

    proc iml;
    A = 6:10;
    B = 1:5;
    anyCommon = any( element(B, A) );
    print anyCommon;
    quit;

    Cheers,
    Rick Wicklin
    Work for SAS; speak for me
    -----------

