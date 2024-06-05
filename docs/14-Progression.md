# Rapport de progression {#progression}



Un des gros problèmes des cours en classes inversées est d'arriver à **stimuler** les étudiants suffisamment pour qu'ils fassent le travail demandé chez eux avant la séance. Livrés à eux-mêmes, seulement un pourcentage trop faible d'entre eux va effectuer complètement la préparation à domicile. Évidemment, c'est également préjudiciable au travail en classe, si ce dernier part du principe que la matière hors présentiel est censée être vue. Et si des travaux de groupe (conseillé) sont prévus, ceux qui n'ont rien fait apparaîtront comme des boulets pour les autres.

\BeginKnitrBlock{warning}<div class="warning">En parallèle au déploiement du matériel d'auto-apprentissage à domicile, nous devons mettre en place des outils pour permettre à l'étudiant de suivre sa progression d'apprentissage, et aussi pour le stimuler dans cet auto-apprentissage.</div>\EndKnitrBlock{warning}

Les mesures qui peuvent être mises en place sont :

- *Pénalisation si le travail n'est pas fait*. Il faut arriver à déterminer ici objectivement si le travail est fait ou pas. Le plus simple est d'interroger les étudiants (mini-interrogation) en début de chaque séance en présentiel, **mais cela prend du temps et n'est pas très constructif** ! Cette approche n'est donc pas conseillée. 

La vidéo suivante [https://www.youtube.com/embed/xp0O2vi8DX4](https://www.youtube.com/embed/xp0O2vi8DX4) explique d'ailleurs pourquoi le coercitif ne fonctionne pas bien et pourquoi il faut plutôt **valoriser le travail effectué** plutôt que de punir s'il n'est pas fait. Il faut aussi **une récompense immédiate**, et favoriser l'**émulation individuelle** (se dépasser soi-même), et **de groupe** (se comparer à la progression générale de la classe).

Les aides et encouragements au travail que nous pouvons mettre en place de manière utile pour faire en sorte que la plus grande fraction possible d'étudiant fasse sérieusement le travail de préparation demandée sont :

- L'*émulation personnelle* au travers de points reçus si l'on fait tous les exercices (les badges, et la ludification en général de l'apprentissage sont également à considérer sérieusement dans ce contexte),

- La *valorisation d'un comportement ou d'un résultat positif au delà de la moyenne*  à l'aide de points bonus (on est toujours dans la gratification et la ludification ici),

- Montrer que nous (en qualité d'enseignants), nous nous *intéressons réellement à leur travail*. Cela tient à des petits détails : des retours sous forme de courts commentaires positifs, des relances, des questions, des encouragements, la reconnaissance de leurs progrès...

- l'*entre-aide*, faire expliquer les points difficiles par un collègue qui les a compris est extrêmement efficace. En plus, cela permet de mitiger un aspect difficile à gérer : la différence de vitesse d'apprentissage entre les "bons" et les "mauvais" étudiants (non, il n'y a pas de bons et de mauvais étudiants, nous le savons, il y a juste des étudiants avec des facultés et des motivations différentes... mais certains sont quand même beaucoup plus lents et **énervants** que d'autres). Au lieu de se tourner les pouces, demander à ceux qui ont fini plus tôt les exercices en séance d'aider les autres permet d'utiliser efficacement le temps et les ressources humaines en classe.

- Les *travaux de groupes*, les *challenges* (par exemple, les étudiants doivent reproduire un graphique difficile), les *compétitions* (une course contre le temps et/ou entre plusieurs groupes pour résoudre un problème) participent efficacement à dynamiser le travail en classe et à varier les activités.

- L'utilisation des *réseaux sociaux* pour communiquer. Les "issues" de GitHub, une fois que les étudiants ont appris à les utiliser, s'avèrent utiles car elles sont disponibles près du dépôt lui-même qui contient les exercices. Mais ce qui fonctionne le mieux (testé en période de confinement Covid-19), c'est un forum d'échange comme [Discord](https://discord.com) parce que beaucoup d'étudiants le connaissent et l'utilisent déjà pour les jeux. Le ton des échanges y est d'ailleurs plus décontracté (parfois nettement plus). C'est un outil efficace pour aider un étudiant qui est bloqué dans sa progression.

- Les étudiants sollicitant une *aide plus directe* apprécient de pouvoir poser leurs questions par email. Afin de pouvoir être plus réactif, nous pouvons mettre en place une adresse mail qui renvoie les messages simultanément à tous les enseignants du cours. Ainsi, les interventions peuvent être partagées et plus rapides.

- Enfin, pour obtenir un *retour plus direct* par rapport aux activités, la plateforme LearnIt::R LRS fournit un **rapport de progression** à l'usage des étudiants. Les actions de l'étudiant dans les exercices H5P, les applications Shiny et les tutoriels learnr étant enregistrées dans notre "LRS" MongoDB, ce rapport de progression présente l'information de manière graphique et résumée afin de pouvoir constater rapidement l'état d'avancement du travail. L'*émulation de groupe* est obtenue en comparant la progression de l'étudiant à la progression générale de la classe.

Afin d'avoir un retour immédiat, ce rapport montre l'évolution en temps réel. Il est envisageable d'associer cette progression à des récompenses sous forme de **badges** lorsque l'étudiant franchit une étape dans son apprentissage (aussi, des badges spéciaux pourraient être associés aux points bonus). Cependnant, cette fonctionnalité n'est pas encore implémentée actuellement. La gestion des badges a été abordée dans la section dédiée à Moodle (voir chapitre \@ref(moodle)).

L'app Shiny de rapport de progression est disponible dans le package [{learnitprogress}](https://github.com/learnitr/learnitprogress). Cette app Shiny peut être intégrée dans le bookdown de votre cours (par exemple à la fin de chaque chapitre) et dans Moodle. Nous utilisons bien entendu les données provenant de notre LRS MongoDB, et en particulier les tables "user", "h5p", "shiny" et "learnr".

## Gestion des utilisateurs

Nous transmettons les informations concernant l'utilisateur à partir de Wordpress ou de Moodle au travers de l'URL qui la lance soit dans une page dédié, soit dans un iframe pour intégrer ce rapport de progression.

TODO: continuer la rédaction de ce chapitre...
