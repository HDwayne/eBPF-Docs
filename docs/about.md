

# Qu'est-ce qu'eBPF ? 

eBPF, ou extended Berkeley Packet Filter, est une technologie qui permet d'exécuter du code de manière sécurisée dans le noyau d'un système d'exploitation ( Windows ou linux ). Initialement développé pour filtrer des paquets réseau, BPF a évolué en eBPF pour devenir une technologie plus générale permettant d'injecter et d'exécuter du code plus complexe et plus diversifié.



# Fonctionnement 

## Structure des programmes eBPF

Dans la majorité des cas, l'utilisation de la technologie eBPF s'articule autour de 2 types de programmes :

### User-space program :
C'est le code qui s'exécute dans l'espace utilisateur ( donc en dehors du noyau ) . Ce programme est responsable de la gestion et de l'interaction avec le code eBPF injecté dans le noyau. Il peut être utilisé pour injecter/retirer le code eBPF, récupérer des résultats via les maps eBPF etc...

Cette partie du code est généralement écrite en C, mais il est possible d'utilisé des API permettant l'utilisation d'un langage avec une plus haute abstraction comme BCC ( BPF compile collection qui utilise Python ) ou encore eBPF-GO ( qui utilise le Go ). En C, certaines librairies existent pour faciliter l'écriture de cette partie du programme comme libbpf. 

### Kernel space program :

C'est le code BPF proprement dit qui est injecté dans le noyau. Ce code est souvent appelé "eBPF program" ou "kernel space program". 
Il peut être injecté dynamiquement dans le noyau et exécuté en réponse à certains événements, tels que l'arrivée de paquets réseau ou l'appel d'une commande système par un processus par exemple. 

En plus d'être obligatoirement écrite en C, cette partie du code est extrêmement contrainte de part le fait qu'elle est exécutée au sein du noyau. Par exemple :
    
- ces programmes ne peuvent pas utiliser de variables non initialisées ou accéder à la mémoire hors limites.

- ces programmes doivent une taille permettant de les injecter au sein du kernel 

- ils doivent avoir une complexité finie

- ils ne doivent pas pouvoir se bloquer de quelques manières que ce soit ( boucles infinies etc.. )

- le processus qui injecte le code eBPF dans le kernel doit posséder les privilèges nécéssaires. 

Ces contraintes sont garantie d'être respecté par le vérifieur eBPF ( voir partie Compilation et vérification )


## Compilation et Vérification

Le code eBPF nécéssite certaines opérations avant d'être injecté au sein du kernel. La première étape est de compiler le fichier sous forme de bytecode ( ELF ) car c'est le type de fichier qui est attendu par le kernel. 

![im1](https://ebpf.io/static/a7160cd231b062b321f2a479a4d0848f/9180b/clang.png "compilation d'un programme eBPF en fichier ELF")

Clang et GCC (depuis la version 10) supporte la compilation des fichiers eBPF.



Par la suite, le fichier ELF passe par un vérifieur qui garantie que le programme tournera correctement au sein du kernel ( le but étant de garantir l'absence de potentiel crashs ou blocages du programme durant son exécution et d'assurer qu'il n'y ait aucune faille de sécurité. Sans cela, exécuter un programme eBPF serait extrêmement risqué ). 

![im2](https://ebpf.io/static/7eec5ccd8f6fbaf055256da4910acd5a/b5f15/loader.png "Processus d'exécution d'un programme eBPF: de la vérification à l'injection au sein du kernel")


Le fichier ELF passe ensuite par un compilateur JIT ( Just in time ) qui le transforme en instruction Assembleur ( spécifique à l'architecture de la machine sur laquelle est le fichier ). Enfin le programme est rattaché à l'événement système qui lui a été associé. 



## Maps eBPF 


Les maps eBPF sont des structures de données utilisés par les programmes eBPF.  Elles servent principalement à stocker et à partager des données entre l'espace utilisateur et l'espace noyau, ainsi qu'entre différentes instances de programmes eBPF ( les autres programmes eBPF peuvent y accéder ainsi que les programmes en dehors du noyau grâce à des appels système ). Pusqu'il peut y avoir des accès concurrent, les opérations sur les map sont atomiques pour préserver la cohérence des données. 

L'information est stockée de manière persistante, ce qui permet d'y accéder en tout temps.

## Evénements 

Chaque programmes eBPF doit être rattaché à un "événement" au sein du noyau. Lorsque cet "événement" survient le programme eBPF est exécuté. Il existe multitude d'événements de différents types:

- Les programmes eBPF peuvent être rattaché à des événements en lien avec l'arrivée et le traitement de paquets réseaux à l'aide du sous-système XDP (eXpress Data Path). ils peuvent également être associés aux événéments liés à la gestion du trafic réseau. 

- Les programmes eBPF peuvent être attachés à certains endroits spécifiques du noyau (tracepoints) pour collecter des informations de traçage en temps réel, permettant ainsi une analyse détaillée de l'exécution du système.

-  Les programmes eBPF peuvent être attachés aux points d'entrée ou de sortie de fonctions du noyau, permettant ainsi de créer des sondes de système (Kprobes) pour le débogage, la surveillance et d'autres tâches.

- Les programmes eBPF peuvent être rattaché au module de sécurité linux (LSM) et influés sur le comportement du kernel. 

- et bien plus encore.....








## Installation

Les outils et librairies qui seront principalement utilisés lors du projet sont :

bpftool : https://github.com/libbpf/bpftool

libbpf : https://github.com/libbpf/libbpf

llvm / clang : https://github.com/llvm/llvm-project (déjà présent sur la majorité des distributions linux)

il est également possible d'utiliser GCC à la place de clang  : https://github.com/gcc-mirror/gcc


## Sources

ebpf.io ( pour les images ) : https://ebpf.io/








