# Documentation Kooc

## - context
## - Kooc
  *   @import
  *   @module
  *   @implementation
  *   @class
  *   Type Checking
  *   Inheritance
  *   [] Expressions
  *   [Symbol Mangling](#symbol-mangling)

###Import
---------
  Permet d'importer un fichier .kh de la même façon que "include" avec une gestion de multiple inclusion.
  @import suivi d'un @from permet aussi de n'importer qu'un module spécifique du fichier.

  * Syntaxe

          @import "filename.kh"
          //  ifndef "FILENAME"
          //    define "FILENAME"
          //    ...
          //    ifndef "FILENAME_MODULENAME"
          //      define "FILENAME_MODULENAME"
          //      ...
          //    endif
          //  endif

          @from "filename.kh" @import "modulename"
          //  ifndef "FILENAME_MODULENAME"
          //    define "FILENAME_MODULENAME"
          //    ...
          //  endif

  * Processus

  Les informations importées sont traduite spécifiquement comme détaillé dans les autres parties.
  Les mots clés ’#ifndef’ ’#define’ ’#endif’ sont ajouté autour de la traduction de la totalité de l'importation et
  autour de chaque module avec le nom correspondant.
  Cette manipulation permet de gèrer des conflits d'inclusion multiple et de pouvoir importer un module séparèment.

###Module
---------



###@class
________
  Le mot clef @class permet de défénir un [ADT][abstract_data_type] instantiable.
  Toute fonction définis dans la région @membre reçoivent un pointeur sur une instance du type de l'ADT en premier parmètre sous le nom de self.
  Le type est définis comme un type utilisateur et transparent.
  Mais les variable d'instance ou de classe sont décorés donc opaque et nécéssite l'utilisation de la syntaxe '[ ]'.

  * Syntaxe
    @class A
    {
      ...
      @member
      {
        ...
      }
      @member ...;
      ...
    }

  * Processus

  * Example
    @class Cla
    {
      int     a;
      float   a;
      // typedef struct _Cla  Cla;
      // typedef struct _ClaClasse  ClaClasse;
      // struct _ClaClasse
      // {
        // int  ~~a~~;
        // float  ~~a~~;
      // }

      void    fct(void);
      // void ~fct~(void);

      @member
      {
        int   a;
        float a;
        void  fct(int);
        void  fct(void);
      }
      @member char tibbers;
      @member char get_tibbers(void);
      // struct _Cla
      // {
      //  int   ~a~;
      //  float   ~a~;
      //  char   ~tibbers~;
      // };
      // void ~fct~(Cla *self, int);
      // void ~fct~(Cla *self);
      // char ~get_tibbers~(Cla *self);
    }


###Inheritance
-----------------
  Est applicable a tout [ADT][abstract_data_type] qui hérite de Object *
  Requeriments: All methods/functions applicable to a *Base* class are also directly applicable to an *Inherited* class
  VTable + RTTI


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

          char    fct(char);
          char    *fct(char, int);
          float     fct(void);

          // char _KCM2liFC5fct1cKC_(char);
          // char *_KCM2liFPC7fct2ciKC_(char, int);
          // float _KCM2liFF4fct0KC_(void);

          @class test
          {
            char    fct(char);
            char    *fct(char, int);
            float     fct(void);

            int     a = 3;
            double  a = 3.2;

            // typedef struct __KCM2liC4testKC_ _KCM2liC4testKC_ ;
            // extern _KCM2liC4testKC_ litestClass;
            // struct __KCM2liC4testKC_
            // {
            //   int  _KCM2liC4testVI1aKC_;
            //   double  _KCM2liC4testVD1aKC_;
            // }

            // char  _KCM2liC4testFC5fct1cKC_(char);
            // char  *_KCM2liC4testFPC6fct2ciKC_(char, int);
            // float  _KCM2liC4testFF4fct0KC_(void);

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
            // int _KCM2liCM4testFMI5fct1iKC_(_KCM2liCM4testKC_ *, int);
          };

          int   i = 42;
          // extern int _KCM2liVI1iKC_
          float i = 4.2;
          // extern int _KCM2liVF1iKC_

        }

#####li.kc

        // int _KCM2liVI1iKC_ = 42
        // float _KCM2liVF1iKC_ = 42
        // _KCM2liC4testKC_ litestClass = {_KCM2liC4testVI1aKC_ = 3, _KCM2liC4testVD1aKC_ = 3.2};
        @implentation li
        {
            char    fct(char c)
            {
              return (c);
            }

            //  char    _KCM2liFF4fct0KC_(char c)
            //  {
            //    return (c);
            //  }

            char    *fct(char c, int i)
            {
              static char sc = c;

              !@(int)[li.i] += i;
              return (&sc);
            }

            //  char    *_KCM2liC4testFPC6fct2ciKC_(char c, int i)
            //  {
            //    static char sc = c;

            //    _KCM2liVI1iKC_ += i;
            //    return (&sc);
            //  }



            float     fct(void)
            {
              return (!@(float)[li.i]);
            }

            // float _KCM2liFF4fct0KC_(void)
            // {
            //  return (_KCM2liVF1iKC_);
            // }

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