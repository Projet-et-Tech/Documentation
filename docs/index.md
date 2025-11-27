# BiblioTech, la documentation de Projet & Tech

Ce site contient toute la documentation de Projet&Tech, l'association de robotique de [Télécom Saint-Étienne](https://www.telecom-st-etienne.fr/).

<script type="text/tikz">
\usepackage{circuitikz}

\begin{circuitikz}[american, voltage shift=0.5]
\draw (0,0)
to[isource, l=$I_0$, v=$V_0$] (0,3)
to[short, -*, i=$I_0$] (2,3)
to[R=$R_1$, i>_=$i_1$] (2,0) -- (0,0);
\draw (2,3) -- (4,3)
to[R=$R_2$, i>_=$i_2$]
(4,0) to[short, -*] (2,0);
\end{circuitikz}

</script>

## À faire

- [ ] Étoffer la page d'accueil
- [ ] Rédiger la documentation électronique
    - [ ] Comment installer et se servir d'Altium ? Évoquer Kicad
    - [ ] Comment on fait les dessins de circuits ? Voir avec CircuiTikz
    - [ ] Bases d'électronique sur l'accueil de l'élec
    - [ ] ...
- [ ] Rédiger la documentation informatique
    - [ ] Comment configurer et se servir d'un Raspberry Pi ?
    - [ ] Comment utiliser les STM ?
    - [ ] Comment concevoir une routine (avec exemple) ?
    - [ ] Bases d'informatique sur l'accueil de l'info
    - [ ] ...
- [ ] Rédiger la documentation en mécanique
    - [ ] Comment installer et utiliser SolidWorks ? (voir FreeCAD)
    - [ ] Bases de mécanique sur l'accueil de la meca
    - [ ] ...
- [ ] Section lore

## Objectifs pour le robot

<!--Liste de tâches avec tous les points à faire du robot et leur état-->

Voir si d'autres sections sont pertinentes, sinon juste présentation en page d'accueil.
