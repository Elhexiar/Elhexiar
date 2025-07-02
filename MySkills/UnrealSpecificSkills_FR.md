# Mes compétences Unreal Engine

J'ai commencé à expérimenter Unreal Engine en tant qu'éditeur de niveau pour Unreal Tournament 4 dans le but de créer des cartes pour jouer avec mes amis et mes professeurs.
J'ai ensuite commencé à expérimenter l'aspect programmation pendant mon temps libre au cours de mes études, car la plupart de nos premières années ont été faites sur Unity ou Phaser.

## Projet de fin d'études

Lors de la sélection des concepts, mon concept a été sélectionné à la condition que je le fusionne avec celui d'une autre étudiante.
Après que notre concept ait été sélectionné comme l'un des concepts du projet de fin d'études, j'ai confié la conception du jeu à un collègue qui pouvait s'occuper de cet aspect à plein temps pour le reste du développement.
Pendant ce temps, je me suis concentré sur l'aspect technique du projet et surtout sur la recherche et le développement.
Nous avons finalement choisi d'opter pour Unreal, l'une des raisons étant mon désir et celui de nombreux membres de l'équipe de sortir du cadre d'Unity et d'apprendre à créer des jeux peux importe le moteur.

Notre jeu étant un ***RTS avec des éléments de jeu de cartes*** était lourd en système, ce qui a nécessité une phase d'architecture bien planifiée et beaucoup de phases de réévaluation.
J'ai décidé de me concentrer sur le rôle de ***Programmeur système*** et ***Programmeur Gameplay***, en déléguant la programmation de l'interface utilisateur et une partie de l'outillage.


Je vais faire de mon mieux pour essayer d'expliquer certains des systèmes que j'ai designé pour notre jeu !

# RTS

Le jeu étant un ***RTS***, il fallait réfléchir à la façon dont nous allions gérer les unités. De plus, nos Game Designers voulaient un ***Système d'escouade*** qui devait être pris en compte.

### Hiérarchie de base

<img width="596" alt="BasicHierarchy" src="https://github.com/user-attachments/assets/906bb0f1-37e1-48b8-9cdb-e3c3b604d63f" />

Chaque **Squad** est composé d'un objet Squad qui va gérer l'initialisation à partir d'un DA (ce qui inclut le spawn d'unités), les ordres à l'échelle de l'escouade, l'espacement de l'escouade.
il contient une liste de toutes les unités qu'il a engendrées pendant sa phase d'initialisation.
Il contient également une file d'attente de commandes et un Behaviour Tree pour gérer le comportement de l'ensemble du groupe, comme l'aggro de l'escouade et la coordination entre toutes les unités.

Chaque **Unité** est engendrée par l'escouade et contient également un Behaviour Tree et une file d'attente de commandes qui est utilisée pour coordonner les commandes de l'ensemble de l'escouade. Elle contient un composant de mouvement qui gère la logique de mouvement, pour des raisons de performance, nous utilisons un mélange du composant de mouvement Floating Pawn et de l'accrochage des unités au Navmesh puisque nous n'avons pas besoin de simuler directement la physique/gravité pour chacune de nos unités.
L'unité la plus proche de la position moyenne de l'escouade est sélectionnée pour devenir son **Sergent**, ce qui gère les collisions Aggro à l'échelle de l'escouade, les mises à jour du brouillard de guerre, ainsi que les vitesses des autres unités pour qu'elles soient synchronisées avec le Sergent et que les mouvements à l'échelle de l'escouade soient plus cohérents.
Si un sergent meurt, l'unité la plus proche devient sergent.

Les **Unités** sont créées en fonction d'une **Structure de formation** donnée par la ressource de données de l'escouade qui contient toutes les informations nécessaires à la création des unités, y compris un tableau 2D des types d'unités et leurs statistiques.

<img width="350" alt="grid" src="https://github.com/user-attachments/assets/126fce57-8b8d-4cb6-9acf-f1356a5fa2d7" />

Nous avons beaucoup de systèmes différents qui doivent communiquer les uns avec les autres, nous avons également beaucoup d'abstraction avec les Behaviour Tree et les systèmes de commande pour les **Unités** et les **Equipes**.
Pour minimiser l'effet des Behaviour Tree sur les performances, nous essayons d'utiliser autant d'event que possible afin de réduire l'impact potentiel sur les performances.


[![RTS Fight](https://img.youtube.com/vi/4p4HSr-GatU/0.jpg)](https://www.youtube.com/watch?v=4p4HSr-GatU)

### Fog of War


[![Fog Of War Showcase Video](https://img.youtube.com/vi/UsWkpj-7U0M/0.jpg)](https://www.youtube.com/watch?v=UsWkpj-7U0M)

Notre jeu avait également besoin à un moment donné d'un brouillard de guerre, que nous avons implémenté avec un RenderTarget sur lequel le sergent de chaque escouade dessine périodiquement une brush circulaire transparente.
Chaque sergent ray trace periodiquement sur un plan sous la carte pour obtenir les coordonnées UV sur lesquelles dessiner.
Cette RenderTarget est ensuite sample par un Decal Actor qui couvre toute la carte.

Quant aux unités ennemies, elles querry périodiquement la RenderTarget à leurs coordonnées UV correspondantes pour savoir si elles sont dans le brouillard de guerre.
Cela nous permet de cacher les maillages et les animations des unités ainsi que le comportement interactif si elles sont dans le brouillard de guerre, ce qui nous permet de gagner un peu en performance.
( Les Querry sont faits de manière asynchrone car la fonction synchrone de base est assez coûteux ).

<img width="600" alt="Fog of War Query Representation" src="https://github.com/user-attachments/assets/215993c9-cbdb-4621-9292-cb76fdcfb00c" />


(Lors des tests, ce système légèrement altéré permettait également de créer une Minimap précise, mais cette fonctionnalité a été supprimée du jeu).

## Système de cartes

![Card](https://github.com/user-attachments/assets/5d2df46a-7566-4b99-9ff1-1abc68c6d55a)


Lorsque j'ai conçu le système de cartes, le rôle et le comportement précis des cartes n'avaient pas encore été entièrement définis.
J'ai opté pour un système de cartes très modulaire afin que tout changement puisse être rapidement ajouté, et pour permettre à nos GD de produire de nouvelles cartes aussi vite que possible dans les phases ultérieures de production et de test.

Les cartes seraient des conteneurs pour des listes d'effets de cartes qui se déclencheraient lors de certains événements ( OnDraw(), OnPlay(), OnDiscard etc... ).
En faisant cela, notre Game Designer serait capable de faire en sorte que n'importe quel effet se produise à n'importe quel déclencheur correspondant.

<img width="600" alt="Fog of War Query Representation" src="https://github.com/user-attachments/assets/e4099d20-2b52-4961-984f-d2220f7fff98" />
<img width="400" alt="Fog of War Query Representation" src="https://github.com/user-attachments/assets/3b386482-9d05-424d-8e6a-165bee03e4ca" />


Chaque **Carte** est composée de OnExecution(), OnFinish() et éventuellement OnTick(). Mais elle contient également une méthode Condition() qui est appelée en premier, d'abord la carte passe en revue chaque effet de carte d'une liste sélectionnée et confirme toutes les conditions. Si toutes sont confirmées, elle exécute alors ses méthodes d'exécution.

![CardEffectCallOrder](https://github.com/user-attachments/assets/7822cb96-b927-42bf-8d3d-82926d88e267)

Cela s'est avéré très utile pour l'intégration des SFX/VFX.
Combiné avec les fonctions utilitaires polyvalentes déjà existantes, cela a permis aux Game Designers et aux Tech Artist d'expérimenter avec le système et d'être plus créatifs, tout en n'ayant pas besoin de faire appel à un programmeur puisque leurs modifications n'affecterait pas la code base.

Par exemple, j'ai ensuite étendu le système pour créer une infobulle lorsqu'un type donné est survolé, je l'ai encore amélioré en permettant à n'importe quel effet de carte de contenir des TooltipWidgets, lors de la construction de la carte, le CardToolTipWidget itère à travers tous les effets de carte pour leur demander des TooltipWidgets afin de construire le CardToolTipWidget.



## Optimisation 

```
void UAC_M_SquadUnitManager::StartAsyncTask()

{
	AsyncTask(ENamedThreads::AnyBackgroundThreadNormalTask, [this]()
	{
		this->DoAsyncTask();

		AsyncTask(ENamedThreads::GameThread, [this]()
		{
			GetWorld()->GetTimerManager().SetTimerForNextTick([this]()
			{
				this->OnAsyncCompleted();
			});
		});
		
	});
}
```

## Une refonte complète très tard dans le développement

Notre jeu a eu beaucoup de problèmes seulement 2 mois avant la date limite, nous n'arrivions pas à trouver comment faire des niveaux intéressants pour notre RTS et nous avons eu beaucoup de mal à ajuster les métriques des unités pour avoir quelque chose d'intéressant et de satisfaisant.
Après de nombreuses tentatives, nous avons fini par abandonner l'aspect RTS car il aurait probablement nécessité trop de travail et d'expérimentation pour aboutir à un résultat satisfaisant nottement au niveau du LD.
Nous avons opté pour un jeu plus proche d'un AutoBattler où le joueur fait apparaître ses unités sur une carte nodale, et où les unités gèrent elles-mêmes leur mouvement en fonction des paramètres des Nodes

Grâce à la façon dont j'ai conçu les systèmes, la transition d'un type de jeu à l'autre s'est faite sans problème. J'ai pu réutiliser mes systèmes de commande d'escouade et de mouvement sans avoir à changer quoi que ce soit, j'ai juste fait en sorte que les Nodes donnent aux unités de nouvelles instructions dans leur SquadCommandQueue lorsqu'elles entrent en collision.

C'est la même chose pour les autres systèmes, puisque j'ai abstrait beaucoup de choses et que je les ai gardées aussi modales que possible, nous avons pu changer notre gameplay de façon assez spectaculaire avec seulement des modifications mineures. Cela nous a beaucoup aidé dans le prototypage et m'a fait gagner beaucoup de temps que j'ai pu utiliser pour améliorer le Game Feel et l'UX (Cette derniere est encore insuffisante pour un projet complet mais pour 3 semaine combiné au debugging reste une fierté).




