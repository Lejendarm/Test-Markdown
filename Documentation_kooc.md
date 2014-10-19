# Documentation KOOC

## context

*	Pourquoi le KOOC

		Nous sommes trois étudiants d'Epitech, ayant été charmés par les languages objets et par
    la piscine Parsing ayant pris lieu le 15/09/2014.
		La curiosité nous a régulièrement poussé à nous poser des questions de notre côté sur le
    fonctionnement des languages objets, c'est donc naturellement que nous avons vu une aubaine
    dans le projet KOOC qui nous permet aujourd'hui de concrétiser cette curiosité, et de pousser
    plus loin notre réflexion.

* Qu'est-ce que le KOOC

	L'objectif du projet KOOC (kind of objective C) est d'implémenter une sur-couche de language
  objet au language C, qui lui donnera un aspect proche du language Objective C.

  Le KOOC permettra de pallier à certains problèmes du C en implémentant des fonctionnalités
  non présentes à la base.

  Le projet permettra par exemple
	- De déclarer des variables ou des fonctions de même nom, mais possédant des types ou des signatures différents.
	- D'importer des fichiers sans avoir besoin de les protéger contre la multiple inclusion à la main.
	- D'implémenter un système de classe, qui n'est pas présent à la base en language C.
  - Et bien d'autres choses.

## KOOC
  *   [Design global](#design-global)
  *   [syntaxes](#syntaxes)
    -   [@import](#@import)
    -   [@module](#@module)
    -   [@implementation](#@implementation)
    -   [@class](#@class)
    -   [Kooc Expressions](#kooc-expressions)
  *   [concepts](#concepts)
    -   [Type Checking](#type-checking)
    -   [Inheritance](#inheritance)
    -   [Symbol Mangling](#symbol-mangling)


# Design global
---------------
  De manière générale l'interpréteur Kooc est un ensemble de plugins. Chaque élement de syntaxe ou qui relève
  des concepts d'interpretations des syntaxes est un plugin. Le kooc utilise donc une classe KoocPluginManager
  qui sert à intégrer et à gérer les interactions entre les différents plugin.

  Chaque Plugin implémente une fonction d'enregistrement qui lui permettera de gérer ses dépendences. Ainsi
  si un plugin nécessite l'utilisation d'autres plugins, il va dans un premier temps vérifier leur présence
  dans le KoocPluginManager, puis il va étendre ce dernier à l'aide d'un Helper pour mettre en place cette
  interaction. Cette étape n'est pas obligatoire car le fonctionnement de chaque Plugin est indépendant.

      Toutefois un nombre de plugins obligatoires est nécéssaire au bon fonctionnement du Kooc, les retirer
      nuira a l'interpretation de fonctionnalités basiques.

  Ce choix de composition permet d'étendre à volonté les syntaxes interpretées par le kooc ainsi que de
  rajouter un nombre de concepts importants sans géner le fonctionnement de base.

  Puis le kooc fait appel à son parseur enrichi par les diférents plugins.

  Le parseur à une mémoire interne des symboles trouvés dans les différents headers kooc.
  Ce qui lui permet de ne pas parser à chaque @import d'un fichier mais de charger directement
  les symboles qui y sont défénis.

![alt text](https://github.com/mohame_e/kooc/ "Logo Title Text 1")

# Syntaxes
----------

 * Les Plugins

  Notre conception du kooc se base sur un fonctionnement à l'aide de plugins.





###@import
---------
  Permets d'importer un fichier .kh de la même façon que "include" avec une gestion de multiples inclusions.
  @import précédé d'un @from permet aussi de n'importer qu'un module spécifique du fichier.

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
 
 On repère le mot clé @import qui a été rajouté par le importPlugin. le plugin lance un parsefile sur le fichier.
 Les mots clés ‘#ifndef‘ ‘#define‘ ‘#endif‘ sont ajouté autour de la traduction de la totalité de l'importation et autour de chaque module.
 Si le fichier a déja été sauvegardé, on ne le traduit pas de nouveau.

###@module
---------
  Le @module situé principalement dans le .kh, marque le début d'une déclaration de module. Toutes les variables et fonctions
  non statiques qui y sont associées sont traitées par le "Symbol Mangling" pour la traduction.

  * Syntaxe

        @module Module_name
        {
          int   i = 42;
          void  fct();
          ...
        }

  * Processus

  Le module définit un ensemble de variables et de fonctions que l'on peut considérer comme étant globale et singleton, Il n'y a qu'une seule instance d'un module.
  Chaque élèment ainsi définit est décoré puis inséré dans le code C;
  /!\ Pour un élèment constant, même si la décoration est différente pour le même élèment non constant, le nom DOIT être différent pour éviter des conflits lors du traitement.

  Pour détailler le processus:

  On charge le ModulePlugin grâce à la fonction load_plugin du pluginManager.
  Le helper associé ModulePlugin repère la déclaration @module pendant le parsing et appelle les plugins dont il a besoin.
  Dans un premier temps le SymbolManglingPlugin va décorer les informations.
  Dans un seconds temp c'est le ModulePlugin qui va traiter les informations précédement décoré.
  A la fin du parsing le buildFile va créer le code C à partir de ces informations.

  * Example

        @module Config
        {
          int     x = 64;
          float   fy = 12.1;

          void  print(char *str);
        }

        // extern int   _KCM6ConfigVI1xKC_ = 64;
        // extern float _KCM6ConfigVF2fyKC_ = 12.1;
        // void         _KCM6ConfigFV8print1pcKC_(char $);

        int main()
        {
          int z;

          z = [Config.x] + 3;
          // z = _KCM6ConfigVI1xKC_ + 3; 
        }

###@implementation
---------
  Le @implementation permet l'implémentation des fonctions associées à un module. le plugin associé devra créer les fonctions que l'utilisateur veut implémenter dans un module.

  * Syntaxe

      @implementation Module_name
      {
          void fct_of_module_name()
          {
            ...
          }
      }

  * Pour détailler le processus:

  On charge le ModulePlugin grâce à la fonction load_plugin du pluginManager.
  Le helper associé ModulePlugin repère la déclaration @module pendant le parsing et appelle les plugins dont il a besoin.
  Dans un premier temps le SymbolManglingPlugin va décorer les informations.
  Dans un seconds temp c'est le ModulePlugin qui va traiter les informations précédement décoré.
  A la fin du parsing le buildFile va créer le code C à partir de ces informations.

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
  Le mot-clef @class permet de définir un [ADT][abstract_data_type] instanciable.
  Toutes fonctions définies dans la région @membre reçoivent un pointeur sur une instance du type de l'ADT en premier paramètre sous le nom de self.
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


###Kooc syntax
--------------

###Type Checking
-----------------
  Le type checking est un système utilisé par le KOOK pour déterminer quelle surcharge de variable ou de fonction est utilisée selon le type ou la signature de la fonction.
	Dans les cas où le type sera impossible à déterminer, la syntaxe "@!("type")" sera utilisée pour forcer le choix de la variable ou fonction voulue. Sinon une erreur de parsing est lancée.

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

  ######main.kc
    @import "c.kh"
    int         main()
    {
      [c.a] = [c fct :c.a];
      // Appele float fct(double);
      // assigne ~c~float~a~;

      [c fct];
      // Appele void   fct(void);

      if ([c fct :c.a]) {}
      // Appele int   fct(int);
      // car le "if" attend un retour.

      @!(double)[c.a] = 4.86;
      // L'utilisation de la syntaxe "@!("type")" est ici obligatoire
      // pour que le compilateur sache si l'affectation doit se faire sur la variable de type double ou float.

      [c fct :c.a]
      // Appele la fonction void fct(int); car la valeur de retour n'est pas utlisée et qu'un paramètre est attendu.
    }

###Inheritance
-----------------
  Est applicable à tout [ADT][abstract_data_type] qui hérite de Object *
  Requeriments: All methods/functions applicable to a *Base* class are also directly applicable to an *Inherited* class
  VTable + RTTI


###Symbol Mangling
-----------------
  *   Si un mot clef du C ou une défintion de type il n'y a pas de symbol mangling généré
  *   prefix de décoration [interdit par le C][1] == '_KC'
  *   puis décration sous paire 'type + len + name'
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



#####li.kh

        @module li
        {
          char    fct(char);
          char    *fct(char, int);
          float   fct(void);

          // char _KCM2liFC5fct1cKC_(char);
          // char *_KCM2liFPC6fct2ciKC_(char, int);
          // float _KCM2liFF4fct0KC_(void);

          int   i = 42;
          float i = 4.2;
          // extern int _KCM2liVI1iKC_
          // extern float _KCM2liVF1iKC_
        }

        @class test
        {
          char    fct(char);
          char    *fct(char, int);
          float     fct(void);

          int     a = 3;
          double  a = 3.2;

          // typedef struct testClass_ testClass;
          // extern testClass _testClass;
          // int  _KCC4testVI1aKC_;
          // double  _KCMC4testVD1aKC_;

          char    fct(char);
          char    *fct(char, int);
          float   fct(void);

          // char  _KCMC4testFC5fct1cKC_(char);
          // char  *_KCMC4testFPC6fct2ciKC_(char, int);
          // float  _KCMC4testFF4fct0KC_(void);

          @member int  fct(int);
          @member
          {
            int   a;
            float a;
            float b;
            int   fct(void);

            // int   fct(int);
            // int _KCCM4testFMI5fct1iKC_();
            // Would produce a conflict error
          }
          // typedef struct test_ test;
          // struct test_
          // {
          //   int _KCCM4testVI1aKC_;
          //   int _KCCM4testVF1aKC_;
          //   int _KCCM4testVF4trollKC_;
          // };
          //
          // test   *new();
          // void  delete(test *);
          //
          // int _KCCM4testFMI4fct0KC_(test *);
          // int _KCCM4testFMI5fct1iKC_(test *, int);
        };


#####li.kc

        // int _KCM2liVI1iKC_ = 42;
        // float _KCM2liVF1iKC_ = 42;
        // _KCC4testVI1aKC_ = 3;
        // _KCC4testVD1aKC_ = 3.2;
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

            //  char    *_KCM2liFPC6fct2ciKC_(char c, int i)
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
        }

        @implementation test
        {

          char    fct(char m)
          {
            return(m);
          }

          float     fct(void)
          {
            return (1);
          };

          char    *fct(char c, int i)
          {
            char  *ret = malloc(sizeof(char) * i + 1);

            while (--i)
              if (ret)
                strncpy(ret, &c, 1);
            return (ret);
          }

          // char  _KCMC4testFC5fct1cKC_(char m)
          // {
          //    return (m);
          // }

          // char  *_KCMC4testFPC6fct2ciKC_(char c, int i)
          // {
          //  char  *ret = malloc(sizeof(char) * i + 1);
          //
          //  while (--i)
          //    if (ret)
          //      strncpy(ret, &c, 1);
          //  return (ret);
          // }

          // float  _KCMC4testFF4fct0KC_(void)
          // {
          //  return (1);
          // }

        }


#####main.kh

        @import "li.kh"

        int     main() {
          test      *l = [li.test new] ;
          int       x;
          char      d;

          x = @!(int)[l fct];
          d = @!(char)[l fct :'c'];
          @!(int)[l.a] = 42;
          @!(float)[l.a] = 4.2;
          return (0);
        }



  [1]: http://en.wikipedia.org/wiki/Name_mangling#Complex_example