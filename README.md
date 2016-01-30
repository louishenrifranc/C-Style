
#  Programme C+++ 

# Header 
  * Pour chaque fichier .cpp associé un fichier .h.
  * Chaque HEADER doit démarre par un 
``` #IFNDEF 	FOO_BAR_BAZ_H_``` si le header se trouve dans foo/src/bar/baz.h 


   * Ordre des inclusions d'header:
     1. header correspondant au .cpp
     2.  C/C++ headers
     3. Autre librairie .h (boost)
     4. Autre header du projet

# Namespace:
- Ne pas mettre de namespace sans nom dans un Header
- Dans un .cpp il est autorisé de mettre des namespace sans nom
- Pas de namespace inline
- Utiliser des raccourcis ```namespace baz = ::foo::bar::baz``` 
- Privilégier les fonctions dans un namespace plutot que des fonctions statiques dans une classe 
```
namespace project{ 
    namespace fs{    // Correct
        void foo1();}
    class fs{
        static void foo1();
    }
}
```
# Variables et Objet
- Declarer les variables/objets au plus prêt de leur utilisation  
- ```std::move``` a utilisé quand l'on veut déplacer un objet
# Classe
- Eviter d'appeler des fonctions virtuelles dans le constructeur
- Faire des initialisation arbitraire des attributs d'un objets
- Il est possible d'appeler plusieurs constructeurs sur un objet:
- Utiliser  ```explicit``` quand cela est nécessaire et proscrire ```implicit```
```
class Object{
	/* explicit */ Object(int i)
}
int k=0;
Object objet = K; // est correcte tant que 
					// le constructeur n'est pas déclaré comme explicit
``` 
* __Les 4 constructeurs utiles__
    - Constructeur par défaut: ne prend pas de paramètres, instancie les attributs
    - Constructeur par copie. Il en est généré un de base par le compilateur. Cependant, il faut le définir si certains objets ont des pointeurs comme attributs. Dans ce cas le constructeur par défaut se contentera de copier le pointeur. Si l'on appelle le destructeur sur l'objet copié après, alors il y aura une erreur dans notre nouvel objet. Voici les 4 syntaxes possibles:
    ```C++
    Class( const Class& other );
    Class( Class& other );
    Class( volatile const Class& other );
    Class( volatile Class& other );
    ```
    - Constructeur de copie par assignement : 
    ```C++
        T& T::operator=(T arg)
        {
            swap(arg); //si la classe T a un attribut n,on a swap(n,arg.n)
           return *this;
        }
    ```
    - Destructeur
    - _C++11 Style_: Constructeur par déplacement. L'objet passé en parametre est déplacé dans le nouveau Objet et le constructeur par défaut. Voici la syntaxe ```Class& C::operator=(C&& other)```. Les 4 étapes sont:
        * supprimer les attributs actuels de la classe
        * copier les attributs de l'objet passé en paramètre, dans la classe
        * mettre les attributs de l'objet passé en parametre à leur valeur par défault
        * renvoyer un pointeur  *this
    - Creer une boucle infinie ```Class( Class other );```

# Inline function
   - Inliner seulement les petites fonctions (tous les accesseurs par exemple)
