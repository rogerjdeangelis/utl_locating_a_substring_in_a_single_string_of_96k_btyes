# utl_locating_a_substring_in_a_single_string_of_96k_btyes
Locating a substring in a single string or file of 96k btyes.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.


    Locating a substring in a single string or file of 96k btyes
       
    see additiona solutions by me and Paul at end

    1. Paul: Additional SAS Solutions  (Paul Dorfman sashole@bellsouth.net)
    2. Roger: WPS/Python and WPS/R ir IML/R solutions


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


 


    *____             _
    |  _ \ __ _ _   _| |
    | |_) / _` | | | | |
    |  __/ (_| | |_| | |
    |_|   \__,_|\__,_|_|

    ;


    Roger,

    Needless to say, I find your approach intriguing, and you've certainly
    provided the POC. My only minor gripes with it are:

    - hard on memory
    - uses a fair amount of hard coding
    - needs more flexible coding to handle arbitrary search strings

    Instead of committing the entire thing to memory, the goal can be
    achieved by searching 2 strings only:

    1. the input record itself
    2. the last N-1 bytes of the prior record concatenated with the
    first N-1 bytes of the current record

    Thus:

    %let RL = 32767 ; * infile record length ;

    filename temp temp lrecl=&RL ;

    data _null_;
      file temp ;
      length str $ &RL ;
      retain src "ABCDEF" ;
      do j = 1 to length (src) ;
        str = repeat (char (src, j), &RL - 1) ;
        put str ;
      end ;
    run ;

    data _null_ ;
      retain SF "EF" VL ; * SF:search-for string, VL: SF system length  ;
      infile temp ;
      input R $char&RL.. ;
      if _n_ = 1 then do ;
        SI = SF || SF     ; * set SI length to double SF ;
        VL = vlength (SF) ; * set VL to SF system length ;
      end ;
      SI = substr (lag (R), &RL - VL + 1) || substr (R, VL - 1) ;
      if find (R, SF) or find (SI, SF) then do ;
        put SF "is found" ;
        stop ;
      end ;
    run ;

    Then there's another, programmatically simpler, unbuffered read input approach you've hinted at:

    data _null_ ;
      retain SF "BBBCCC" SI ; * SF:search-for, SI:search-in ;
      if _n_ = 1 then SI = translate (SF, "", SF) ;
      infile temp recfm = N ;
      input C $1. ;
      if C in ("0d"x, "0a"x) then delete ;
      SI = ifC (length (SI) < vlength (SI), cats (SI, C), cats (substr (SI, 2), C)) ;
      if SI = SF then do ;
        put SF "is found" ;
        stop ;
      end ;
    run ;

    Of course, the method is not devoid of its own shortcomings:

    - the hard coded CRLF characters will have to be re-coded under a different OS
    (though it can be made dependent on the system macro variable, such as  SYSSCP)
    - since it reads one byte at a time, it's likely to work slower

    Best regards,
    Paul Dorfman


    *____
    |  _ \ ___   __ _  ___ _ __
    | |_) / _ \ / _` |/ _ \ '__|
    |  _ < (_) | (_| |  __/ |
    |_| \_\___/ \__, |\___|_|
                |___/
    ;

    *            _   _
     _ __  _   _| |_| |__   ___  _ __
    | '_ \| | | | __| '_ \ / _ \| '_ \
    | |_) | |_| | |_| | | | (_) | | | |
    | .__/ \__, |\__|_| |_|\___/|_| |_|
    |_|    |___/
    ;

    %utl_submit_wps64("
    options set=PYTHONHOME 'C:\Progra~1\Python~1.5\';
    options set=PYTHONPATH 'C:\Progra~1\Python~1.5\lib\';
    libname sd1 'd:/sd1';
    proc python;
    submit;
    f = open('d:/txt/have.txt','r');
    string = f.read();
    print (string.find('AB'));
    endsubmit;
    run;quit;
    ");


    0 offset;
    32766

    *____
    |  _ \
    | |_) |
    |  _ <
    |_| \_\

    ;

    %utl_submit_wps64('
    options set=R_HOME "C:/Program Files/R/R-3.3.2";
    proc r;
    submit;
    source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
    library(readr);
    library(stringr);
    mystring <- read_file("d:/txt/have.txt");
    grepl("AB", mystring);
    str_locate(mystring, "AB");
    endsubmit;
    run;quit;
    ');

    The WPS System

    1 Offset

    [1] TRUE
         start   end
    [1,] 32767 32768



