--- 
title: "LearnIt::R plateforme pédagogique pour l'apprentissage de R"
author: "Philippe Grosjean & Guyliann Engels"
date: "2024-06-23"
site: bookdown::bookdown_site
output:
  bookdown::gitbook:
    info: yes
    includes:
      in_header: header.html
      after_body: footer.html
  bookdown::pdf_book:
    includes:
      in_header: preamble.tex
    latex_engine: xelatex
    citation_package: natbib
    number_sections: yes
    fig_caption: yes
    keep_tex: no
  bookdown::epub_book:
    stylesheet: style.css
    number_sections: yes
    fig_caption: yes
documentclass: book
bibliography: [book.bib, packages.bib]
biblio-style: apalike
link-citations: yes
github-repo: learnitr/learnitrdoc
url: 'https\://learnitr.github.io/learnitrdoc/'
description: "Documentation pour les outils pédagogiques LearnIt::R."
cover-image: "images/learnitr_800.png"
---

# Préambule {-}



\BeginKnitrBlock{warning}<div class="warning">
**Cet ouvrage est en cours d'écriture. Plusieurs chapitres doivent encore être écrits ou remaniés. Une version anglaise doit également être rédigée.** Cependant, les informations qu'il contient sont déjà utilisables pour mettre en place certains des outils documentés ici. N'hésitez pas à nous contacter ([Philippe Grosjean](mailto:Philippe.Grosjean@umons.ac.be) ou [Guyliann Engels](mailto:Guyliann.Engels@umons.ac.be)) pour plus d'information ou pour discuter de l'intégration de tel ou tel outil pédagogique dans vos propres cours.
</div>\EndKnitrBlock{warning}

![](images/learnitr_800.png)

Cet ouvrage présente en détails la **plateforme pédagogique LearnIt::R** spécifique pour l'apprentissage de R, des statistiques et de la science des données en général. Il est dérivé de notes relatives au développement, à la maintenance et à la création de contenu sur notre plateforme autour du package R {learnitdown} que nous avons créé à l'Université de Mons en Belgique pour nos [cours de Science des Données Biologiques](https://wp.sciviews.org). Désolé, donc, si vous y lirez de temps en temps des informations qui ne sont utiles **que** dans le cadre spécifiquement de ces cours-là. Notre objectif à terme est de migrer progressivement cet ouvrage et les outils qu'il décrit vers un usage plus large pour permettre de déployer une plateforme pédagogique similaire dans un autre contexte...

![](images/front-cover.png)

_Le matériel dans cet ouvrage est distribué sous licence [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.fr)._

<details>
<summary>Détails concernant le système utilisé pour compiler ce bookdown...</summary>

##### Information système {-}


```r
sessioninfo::session_info()
```

```
# ─ Session info ───────────────────────────────────────────────────────────────
#  setting  value
#  version  R version 4.2.3 (2023-03-15)
#  os       macOS 14.5
#  system   aarch64, darwin20
#  ui       X11
#  language (EN)
#  collate  en_US.UTF-8
#  ctype    en_US.UTF-8
#  tz       Europe/Brussels
#  date     2024-06-23
#  pandoc   3.1.11 @ /Applications/RStudio.app/Contents/Resources/app/quarto/bin/tools/aarch64/ (via rmarkdown)
# 
# ─ Packages ───────────────────────────────────────────────────────────────────
#  package     * version date (UTC) lib source
#  bookdown      0.39    2024-04-15 [1] CRAN (R 4.2.3)
#  bslib         0.4.2   2022-12-16 [2] RSPM (R 4.2.0)
#  cachem        1.0.7   2023-02-24 [2] RSPM (R 4.2.3)
#  cli           3.6.1   2023-03-23 [2] RSPM (R 4.2.0)
#  digest        0.6.31  2022-12-11 [2] RSPM (R 4.2.0)
#  evaluate      0.20    2023-01-17 [2] RSPM (R 4.2.3)
#  fastmap       1.1.1   2023-02-24 [2] RSPM (R 4.2.0)
#  htmltools     0.5.5   2023-03-23 [2] RSPM (R 4.2.3)
#  jquerylib     0.1.4   2021-04-26 [2] RSPM (R 4.2.0)
#  jsonlite      1.8.4   2022-12-06 [2] RSPM (R 4.2.0)
#  knitr         1.42    2023-01-25 [2] RSPM (R 4.2.3)
#  R6            2.5.1   2021-08-19 [2] RSPM (R 4.2.0)
#  rlang         1.1.1   2024-01-06 [2] Github (r-lib/rlang@564f176)
#  rmarkdown     2.21    2023-03-26 [2] RSPM (R 4.2.3)
#  rstudioapi    0.14    2022-08-22 [2] RSPM (R 4.2.0)
#  sass          0.4.5   2023-01-24 [2] RSPM (R 4.2.3)
#  sessioninfo   1.2.2   2021-12-06 [1] CRAN (R 4.2.0)
#  xfun          0.44    2024-05-15 [1] CRAN (R 4.2.3)
#  yaml          2.3.7   2023-01-23 [2] RSPM (R 4.2.0)
# 
#  [1] /Users/phgrosjean/Library/R/arm64/4.2/library
#  [2] /Library/Frameworks/R.framework/Versions/4.2-arm64/Resources/library
# 
# ──────────────────────────────────────────────────────────────────────────────
```

</details>
