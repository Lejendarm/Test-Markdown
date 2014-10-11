# Documentation KOOC

## - context

	*	Pourquoi le KOOC

			Nous sommes trois étudiants d'Epitech, ayant été charmés par les languages objets et par la piscine Parsing ayant pris lieu le 15/09/2014.
			La curiosité nous a régulièrement poussé à nous poser des questions de notre côté sur le fonctionnement des languages objets,
			c'est donc naturellement que nous avons vu une aubaine dans le projet KOOC qui nous permet aujourd'hui de concrétiser cette curiosité, et de pousser plus loin notre réflexion.

	* Qu'est-ce que le KOOC

			L'objectif du projet KOOC (kind of objective C) est d'implémenter une sur-couche de language objet au language C, qui lui donnera un aspect proche du language Objective C.
			Le KOOC permettra de pallier à certains problèmes du C en implémentant des fonctionnalités non présentes à la base.
			Le projet permettra par exemple :
				- De déclarer des variables ou des fonctions de même nom, mais possédant des types ou des paramètres différents.
				- D'importer des fichiers sans avoir besoin de les protéger contre la multiple inclusion à la main.
				- D'implémenter un système de classe, qui n'est pas présent a la base en language C.
			Et bien d'autres choses.

## - KOOC
  *   @import
  *   @module
  *   @implementation
  *   @class
  *   Type Checking
  *   Inheritance
  *   [] Expressions
  *   [Symbol Mangling](#symbol-mangling)

###@import
---------
  Permets d'importer un fichier .kh de la même façon que "include" avec une gestion de multiples inclusions.
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
  Les mots clés ‘#ifndef‘ ‘#define‘ ‘#endif‘ sont ajouté autour de la traduction de la totalité de l'importation et
  autour de chaque module avec le nom correspondant.
  Cette manipulation permet de gèrer des conflits d'inclusion multiple et de pouvoir importer un module séparèment.

###@module
---------
  Le @module situé principalement dans le .kh, marque le début d'une déclaration de module. Toutes les variables et function
  non statique qui y sont associées sont traité par le "Symbol Mangling" pour la traduction.

  * Syntaxe

        @module Module_name
        {
          int   i = 42;
          void  fct();
          ...
        }

  * Processus

  Le module définit un ensemble de variable et de fonctions que l'on peu considérer comme étant globale et singleton, Il n'y a qu'une seule instance d'un module.
  Chaque élèment ainsi définit est décoré puis inséré dans le code C;
  /!\ Pour un élèment constant, même si la décoration est différente pour le même élèment non constant, le nom DOIT être différent pour éviter des conflits lors du traitement.

  * Example

        @module Config
        {
          int   x = 64;
          int   y = 128;
          
          void  print_on_win(char *str);
        }

        int main()
        {
          printf("Windows width:%d height:%d\n", [Config.x], [Config.y]);
        }

###@implementation
---------
  Le @implementation permet l'implementation des fonctions associées à un module.

  * Syntaxe

      @implementation Module_name
      {
          void fct_of_module_name()
          {
            ...
          }
      }

  * Processus
  L'implementation fait le liens avec les fonctions déclarées dans le module.
  Elle contient le code de ces fonctions;

  * Example

      @module Joke
      {
        void toctoc();
      }

      @implementation Joke
      {
        void toctoc(int i)
        {
          while (i > 0)
          {
            printf("JOKE\n");
            i--;
          }
        }
      }

      int main()
      {
        [Joke toctoc :5];
      }

###@class
________
  Le mot-clef @class permet de définir un [ADT][abstract_data_type] instantiable.
  Toute fonction définie dans la région @membre reçoivent un pointeur sur une instance du type de l'ADT en premier paramètre sous le nom de self.
  Le type est défini comme un type utilisateur et transparent.
  Mais les variables d'instance ou de classe sont décorées donc opaques et nécessitent l'utilisation de la syntaxe "[ ]".

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


###Type Checking
-----------------
	Le type checking permet de faire la différence entre des variables ou fonctions de même nom mais de types, ou prenant des paramètres, différents.
	Le type checking du projet KOOC sera relativement stricte, tout ce qui ne sera pas explicite génèrera une erreur de compilation.
	Dans les cas où le type sera impossible à déterminer, la syntaxe "@!("type")" sera utilisée pour forcer le choix de la variable ou fonction voulue.

	* Example
	
	#####c.kh

			@module c
			{
        int     a;
        float   a;
        double  a;
        void    fct(int);
        void    fct(void);
        int     fct(char);
        float   fct(double);
			}

  ######c.kc
    @import "c.kh"
    int         main()
    {
      [c.a] = [c fct :c.a]; // Appellera : float   fct(double);
      [c fct]; // Appellera : void   fct(void);
      if ([c fct :c.a]) {} // Appellera : int   fct(int); car le "if" attend un retour.
      @!(double)c.a = 4.86; // L'utilisation de la syntaxe "@!("type")" est ici obligatoire pour que le compilateur sache si l'affectation doit se faire sur la variable de type double ou float.
      [c fct :c.a] // Appellera la fonction void  fct(int); car l'appel est en dehors de tout contexte.
    }

###Inheritance
-----------------
  Est applicable à tout [ADT][abstract_data_type] qui hérite de Object *
  Requeriments: All methods/functions applicable to a *Base* class are also directly applicable to an *Inherited* class
  VTable + RTTI


###Symbol Mangling
-----------------
  *   prefix de décoration [interdit par le C][1] == '_KC'
  *   puis décration sous paire '\<type + len, name\>'
      1.  type
          V pour "variable" suivit de l'inital du type si de type scalaire sinon nom du type
          F \(+M si méthode\) pour "fonction" suivit de l'inital du type de valeur de retour si de type scalaire sinon nom du type
          M pour "module"
          C pour "class" CM pour "class member"
      2. len
          Longueur du name
      3. name
          si variable nom de la variable
          si fonction nom de la fonction suivit d'un nombre de variable et types

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
              float b;
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