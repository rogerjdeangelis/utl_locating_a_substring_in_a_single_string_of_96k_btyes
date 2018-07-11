# utl_locating_a_substring_in_a_single_string_of_96k_btyes
Locating a substring in a single string or file of 96k btyes.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.


    Locating a substring in a single string or file of 96k btyes

    see github
    https://tinyurl.com/yagy8ecm
    https://github.com/rogerjdeangelis/utl_locating_a_substring_in_a_single_string_of_96k_btyes

    This is a proof of concept. There are other ways to do this, ie stream input, but
    this maps the entire string to continuous memory.

    The SAS datastep only supports strings of 32,767 bytes or less.

    The code below finds the location of the string 'ab' in a 96k string;

    Failed in WPS
    %%%%%%%%%%%%%%%%%%%%
    Assertion failed: Not for a temporary array at file:
      j:\wps\prod\3.3\b\win-x64\src\datastep\SymbolTable.hpp line 170
    %%%%%%%%%%%%%%%%%%%%
    Crash dump generated in C:\Users\beast\AppData\Local\World Programming\WPS\3\Crash Dumps\wps-20180711-113842.dmp


    INPUT (we will read the file as one 96k string - XML/JSON/HTML?)
    ============================================================

      d:/txt/have.txt (stream of text)

                       98,301 Bytes String

          32,767           32,767        32,767
      |                 |            |            |
      AAA .........AAAAABBB.....BBBBBBCCCC......CCC


    EXAMPLE OUTPUT
    --------------

     TXT and Its location in string (in location 32,767 and 32,768)

     WORK.WANT total obs=1

     LOCATION    TXT

       32767     AB


    PROCESS
    ========

    data WANT;

     length stra strb strc $32767;

     infile "d:/txt/have.txt" lrecl=32767 recfm=f;

     input stra;
     input strb;
     input strc;

     array strs[98301] $1 _temporary_;

     call pokelong (stra, addrlong (strs[1])) ;
     call pokelong (strb, addrlong (strs[32768])) ;
     call pokelong (strc, addrlong (strs[65534])) ;

     do location=1 to 98300;
        txt=peekclong (addrlong (strs[location]) , 2);
        if txt="AB" then leave;
     end;
     put location= txt=;
     keep location txt;

    run;quit;

    OUTPUT
    ======

     WORK.WANT total obs=1

     LOCATION    TXT

       32767     AB

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    data _null_;

      file "d:/txt/have.txt" lrecl=32767 recfm=f;

      length stra strb strc $32767;

      stra=repeat('A',32767);
      strb=repeat('B',32767);
      strc=repeat('C',32767);

      put stra;
      put strb;
      put strc;

    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    %utl_submit_wps64('
    libname wrk sas7bdat "%sysfunc(pathname(work))";
    data wrk.want;

     length stra strb strc $32767;

     infile "d:/txt/have.txt" lrecl=32767 recfm=f;

     input stra;
     input strb;
     input strc;

     array strs[98301] $1 _temporary_;

     call pokelong (stra, addrlong (strs[1])) ;
     call pokelong (strb, addrlong (strs[32768])) ;
     call pokelong (strc, addrlong (strs[65534])) ;

     do location=1 to 98300;
        txt=peekclong (addrlong (strs[location]) , 2);
        if txt="AB" then leave;
     end;
     put location= txt=;
     keep location txt;

    run;quit;
    ');
