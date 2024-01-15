

# Qu'est-ce qu'eBPF ? 

eBPF, ou extended Berkeley Packet Filter, est une technologie qui permet d'exécuter du code de manière sécurisée dans le noyau d'un système d'exploitation ( Windows ou linux ). Initialement développé pour filtrer des paquets réseau, BPF a évolué en eBPF pour devenir une infrastructure plus générale permettant d'injecter et d'exécuter du code plus complexe et plus diversifié.



# Fonctionnement 

## Structure des programmes eBPF

Dans la majorité des cas, l'utilisation de la technologie eBPF s'articule autour de 2 types de programmes :

User-space program :

    C'est le code qui s'exécute dans l'espace utilisateur ( donc en dehors du noyau ) . Ce programme est responsable de la gestion et de l'interaction avec le code eBPF injecté dans le noyau. Il peut être utilisé pour injecter/retirer le code eBPF, récupérer des résultats via les maps eBPF etc...

    Cette partie du code est généralement écrite en C, mais il est possible d'utilisé des API permettant l'utilisation d'un langage avec une plus haute abstraction comme BCC ( BPF compile collection qui utilise Python ) ou encore eBPF-GO ( qui utilise le Go ). En C, certaines librairies existent pour faciliter l'écriture de cette partie du programme comme libbpf. 

Kernel space program :

    C'est le code BPF proprement dit qui est injecté dans le noyau. Ce code est souvent appelé "eBPF program" ou "kernel space program". 
    Il peut être injecté dynamiquement dans le noyau et exécuté en réponse à certains événements, tels que l'arrivée de paquets réseau ou l'appel d'une commande système par un processus par exemple. 

    En plus d'être obligatoirement écrite en C, cette partie du code est extrêmement contrainte de part le fait qu'elle est exécutée au sein du noyau. Par exemple :
        - ces programmes ne peuvent pas utiliser de variables non initialisées ou accéder à la mémoire hors limites.

        - ces programmes doivent une taille permettant de les injecter au sein du kernel 

        - ils doivent avoir une complexité finie

        - ils ne doivent pas pouvoir se bloquer de quelques manières que ce soit ( boucles infinies etc.. )

        - le processus qui injecte le code eBPF dans le kernel doit posséder les prvilèges nécéssaires. 

    Ces contraintes sont garantie d'être respecté par le vérifieur eBPF ( voir partie Compilation et exécution)


## Compilation et exécution

*TODO*


## Maps eBPF 

*TODO*

## Installation
/*TODO*/ (bpftool libbpf , llvm, clang )



