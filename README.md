# utl-intersection-of-words-in-two-lists
Common words in two lists  
    %let pgm=utl-intersection-of-words-in-two-lists;

    github
    https://tinyurl.com/v8ratsjn
    https://github.com/rogerjdeangelis/utl-intersection-of-words-in-two-lists


    Problem

    INTERSECTION OF WORDS IN TWO LISTS;
    Common words in two lists
                                         |   Rules
                                                                      |   Common Words
      SRC                              TRG                            |   RESULT
                                                                      |
      one two three four               two three                      |   two three
      five six seven eight             one two three four             |
      one two three four five          five                           |   five
      roger suzie roger suzie roger    roger roger suzie suzie roger  |   roger suzie
      sas wps                          sas wps                        |   sas wps
      taco taco taco                   taco                           |   taco
      taco                             taco taco                      |   taco

     Four SOLUTIONS

        1. SAS macro
        2. SAS error detect

    /*
     _                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    * find common words in source and target list;
    libname sd1 "d:/sd1";
    data sd1.have;
      informat src trg $200.;
      infile cards delimiter=',';
      input src trg;
    cards4;
    one two three four,two three
    five six seven eight,one two three four
    one two three four five,five
    roger suzie roger suzie roger,roger roger suzie suzie roger
    sas wps,sas wps
    taco taco taco,taco
    taco,taco taco
    ;;;;
    run;quit;

    Up to 40 obs from SD1.HAVE total obs=7 25SEP2022:08:10:46

    Obs    SRC                              TRG

     1     one two three four               two three
     2     five six seven eight             one two three four
     3     one two three four five          five
     4     roger suzie roger suzie roger    roger roger suzie suzie roger
     5     sas wps                          sas wps
     6     taco taco taco                   taco
     7     taco                             taco taco

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    Up to 40 obs from WANT_DATASTEP total obs=7 25SEP2022:08:22:40

    Obs    RESULT         SRC                              TRG

     1     two three      one two three four               two three
     2                    five six seven eight             one two three four
     3     five           one two three four five          five
     4     roger suzie    roger suzie roger suzie roger    roger roger suzie suzie roger
     5     sas wps        sas wps                          sas wps
     6     taco           taco taco taco                   taco
     7     taco           taco                             taco taco


    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
     _
    / |    ___  __ _ ___   _ __ ___   __ _  ___ _ __ ___
    | |   / __|/ _` / __| | `_ ` _ \ / _` |/ __| `__/ _ \
    | |_  \__ \ (_| \__ \ | | | | | | (_| | (__| | | (_) |
    |_(_) |___/\__,_|___/ |_| |_| |_|\__,_|\___|_|  \___/

    */

    proc datasets lib=work mt=data kill nodetails nolist;
    run;quit;

    * only need if repeated interactive runs ;
    %symdel arg_src arg_trg arg_result / nowarn;

    libname sd1 "d:/sd1";

    options sasautos=("c:/oto" sasautos);
    data want_macro;
      %inc "c:/oto/utl_intersectWords.sas";
      set sd1.have;

      %utl_intersectWords(
          src      /* source list of words  */
         ,trg      /* target list of words  */
         ,result   /* returned common words */
      );

    proc print data=want_macro;
    run;quit;

    /*___                                                     _      _            _
    |___ \     ___  __ _ ___    ___ _ __ _ __ ___  _ __    __| | ___| |_ ___  ___| |_
      __) |   / __|/ _` / __|  / _ \ `__| `__/ _ \| `__|  / _` |/ _ \ __/ _ \/ __| __|
     / __/ _  \__ \ (_| \__ \ |  __/ |  | | | (_) | |    | (_| |  __/ ||  __/ (__| |_
    |_____(_) |___/\__,_|___/  \___|_|  |_|  \___/|_|     \__,_|\___|\__\___|\___|\__|
    */

    * run with missing arguments;
    data tst;
      set sd1.have;
      %utl_intersectWords;
    run;quit;

    **** Please Provide blank separated source word list ****
    **** Please Provide blank separated target word list ****
    **** Please Provide return variable  ****

    +---------------------------------------------+
    |                                             |
    | Sample Call                                 |
    |                                             |
    | %utl_intersectWords(src,trg,result)         |
    |                                             |
    | %utl_intersectWords(                        |
    |    src     ==>  source list of words        |
    |   ,trg     ==>  target list of words        |
    |   ,result  ==>  common words                |
    | )                                           |
    |                                             |
    +---------------------------------------------+

    ERROR: Stoped incorrect invocation of macro utl_intersectWords


    /*   _                  _               _
     ___| |_ __ _ _ __   __| | __ _ _ __ __| |  _ __ ___   __ _  ___ _ __ ___
    / __| __/ _` | `_ \ / _` |/ _` | `__/ _` | | `_ ` _ \ / _` |/ __| `__/ _ \
    \__ \ || (_| | | | | (_| | (_| | | | (_| | | | | | | | (_| | (__| | | (_) |
    |___/\__\__,_|_| |_|\__,_|\__,_|_|  \__,_| |_| |_| |_|\__,_|\___|_|  \___/

    */


    %macro utl_intersectWords(
        arg_src      /* source list of words */
       ,arg_trg      /* target list of words */
       ,arg_result   /* common words         */
       ) /des= "Create a list of distinct common words contained in two lists";

       %local res ;

       /*--- check arguments                                                         --*/

       put ;
       put " %sysfunc(ifc(%sysevalf(%superq(arg_src)=,boolean),    **** Please Provide blank separated source word list ****,))" ;
       put " %sysfunc(ifc(%sysevalf(%superq(arg_trg)=,boolean),    **** Please Provide blank separated target word list ****,))" ;
       put " %sysfunc(ifc(%sysevalf(%superq(arg_result)=,boolean) ,**** Please Provide return variable                  ****,))" ;

        %let res= %eval
        (
            %sysfunc(ifc(%sysevalf(%superq(arg_src    )=,boolean),1,0))
          + %sysfunc(ifc(%sysevalf(%superq(arg_trg    )=,boolean),1,0))
          + %sysfunc(ifc(%sysevalf(%superq(arg_result )=,boolean),1,0))
        ) ;

        %put &=res;

         /*-- missing one or more arguments                                          --*/

         %if &res ne 0 %then %do ;

           put ;
           put '+---------------------------------------------+';
           put '|                                             |';
           put '| Sample Call                                 |';
           put '|                                             |';
           put '| %utl_intersectWords(src,trg,result)         |';
           put '|                                             |';
           put '| %utl_intersectWords(                        |';
           put '|    src     ==>  source list of words        |';
           put '|   ,trg     ==>  target list of words        |';
           put '|   ,result  ==>  common words                |';
           put '| )                                           |';
           put '|                                             |';
           put '+---------------------------------------------+';
           put ;

           put 'ERROR: macro utl_intersectWords stopped due to incorrect invocation';

           stop;

           run;quit;

        %end;

        /*-- all arguments ok                                                        --*/
        /*-- start common variable code                                              --*/
        %else %do;

            /*-- none of the variables created by this macro will be in the output   --*/
            /*-- temporary arrays are not written to output datasets                 --*/

            array __c[2] $ 200 _temporary_;
            array __n[1] 8     _temporary_;  /*-- for saving _n_                     --*/

            /*-- unfortunately the do statement does not support _temporary_ arrays  --*/
            /*-- save currenr temprary _n_ value to retore later                     --*/
            __n[1] = _n_;

            /*--  hold _temporary_ intermediate values
               __c[1] temp storage for single words from source
               __c[2] remp storage for intermediate results
            --*/

            /*-- cannot use a temp variable in do statement                          --*/
            /*-- iterate over each word in source string                             --*/

            do  _n_ = 1 to  countw(&arg_src)   ;

                __c[1] = scan(&arg_src,_n_) ; /*-- get each word in the source list  --*/

                /*-- if the source word in target string then proceed                --*/
                if index(&arg_trg,strip(__c[1])) > 0 then do ;

                     /* -- do not add word to result list if already present         --*/
                     if index(__c[2],strip(__c[1])) = 0 then
                        __c[2]=catx(" ",__c[2],__c[1]);

                end ; /* adding one word that is not aready in result list           --*/

            end; /* end adding distinct words in source and target to result list    --*/

            &arg_result      = __c[2] ;

            __c[2] = "";

            * restore _n_; /*-- restore original bact to calling program             --*/
            _n_ = __n[1] ;

        %end;

    %mend utl_intersectWords;
    ;;;;
    run;quit;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
