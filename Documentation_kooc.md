# Documentation Kooc

## - context
## - Kooc
  *   #### @import
  *   #### @module
  *   #### @implementation
  *   #### @class
  *   #### Type Checking
  *   #### Inheritance
  *   #### [] Expressions
  *   #### [Symbol Mangling](#symbol-mangling)


###Symbol Mangling
-----------------
  *   prefix de décoration [interdit par le C][1] == '_KC'
  *   puis décration sous paire '\<type + len, name\>'
      1.  type
          V pour variable suivit de l'inital du type si de type scalaire sinon nom du type
          F \(+M si méthode\) pour fonction suivit de l'inital du type de valeur de retour si de type scalaire sinon nom du type
          M pour module
          C pour class CM pour class member
      2. len
          Longueur du name
      3. name
          si variable nom de la variable
          si fonction nom de la fonction suvit d'un nombre de variable et types

  *   puis décoration de fin == 'KC_'
  ainsi




#####li.kh

        @module li
        {
          @class test
          {
            char    fct(char m);
            char    *fct(char, int);
            int     fct(void);
            int     a = 3;
            double  a = 3.2;
            // typedef struct __KCM2liC4testKC_ _KCM2liC4testKC_ ;
            // extern _KCM2liC4testKC_ litestClass;
            // struct __KCM2liC4testKC_
            // {
            //   int  _KCM2liC4testVI1aKC_;
            //   double  _KCM2liC4testVD1aKC_;
            // }

            // char  _KCM2liC4testFC5fct1cKC_();
            // char  *_KCM2liC4testFPC6fct2ciKC_();
            // char  _KCM2liC4testFI4fct0KC_();

            @member int  fct(int);
            @member
            {
              int   a;
              float a;
              float troll;
              int   fct(void);

              // int   fct(int);
              // int _KCM2liCM4testFMI5fct1iKC_();
              // Would produce a conflict error
            }
            // typedef struct __KCM2liCM4testKC_ _KCM2liCM4testKC_ ;
            // struct __KCM2liCM4testKC_
            // {
            //   int _KCM2liCM4testVI1aKC_;
            //   int _KCM2liCM4testVF1aKC_;
            //   int _KCM2liCM4testVF4trollKC_;
            // }
            //
            // _KCM2liCM4testKC_   *new();
            // void  delete(_KCM2liCM4testKC_   *);
            //
            // int _KCM2liCM4testFMI4fct0KC_(_KCM2liCM4testKC_ *);
            // int _KCM2liCM4testFMI5fct1iKC_(_KCM2liCM4testKC_ *);
          };

          int   i = 42;
          // extern int _KCM2liVI1iKC_
          float i = 4.2;
          // extern int _KCM2liVF1iKC_
        }

#####li.kc

        @implentation li
        {
          @implementation test
          {

            char    fct(char m)
            {
              return(m);
            }

            int     fct(void)
            {
              return (1);
            };
          }
        }

#####main.kh

        @import "li.kh"

        int     main() {
          [li.test] *l = [li.test new] ;
          int       x;
          char      d;

          x = @!(int)[l fct];
          d = @!(char)[l fct :'c'];
          @!(int)[l.a] = 42;
          @!(float)[l.a] = 4.2;
          return (0);
        }


  [1]: http://en.wikipedia.org/wiki/Name_mangling#Complex_example