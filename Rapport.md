﻿# Partie informatique

## 1. Objectif général

Cette partie du projet vise à gérer la logique de l’énigme via un microcontrôleur STM32. Le programme contrôle une LED affichant un mot en morse, lit un phototransistor pour vérifier la position d’un "codex", et commande un moteur (via PWM) permettant l’ouverture d’une boîte contenant une clé.

## 2. Configuration matérielle

Avant le développement logiciel, une configuration matérielle du microcontrôleur est nécessaire en fonction des besoins du système.

### Ressources utilisées :

#### Configuration générale

| Fonction | Broche | Configuration | Détail |
|----------------------------------|----------------------|---------------|------------------------------|
| LED morse | PA10 | GPIO_Output | Contrôle de l'affichage en morse  |
| Bouton de déclenchement | PA4 | GPIO_Input | Détection de l'appui utilisateur |
| Lecture du phototransistor | PA3 | ADC | Vérification de la résolution du codex |
| Commande du moteur | PA9 | PWM | Contrôle de l'ouverture de la boîte |

#### Configuration des composants

Il y a deux timers dans le projet. Un (tim2) pour faire fonctionner le code. Nous verrons son utilité dans la partie "Structure logicielle". Le second timer (tim1) génère le signal PWM permettant de contrôler les moteurs. 

Le timer 2 est paramétré pour généré une interruption toute les millisecondes (PSC = 3 et ARR = 9999). Ce qui est largement suffisant pour notre programme qui dépend surtout de l'interaction avec l'utilisateur.

Tandis que le timer 1 est configuré pour générer un signal PWM sur le channel 2 (PA9) pour des raisons de débogage (PA8 soit channel 1 est aussi utilisé par la carte dans le mode debug). Par défaut, le timer possède un PSC de 3 et un ARR de 19999. Et, l'auto reload preload est activé.

L'adc est simplement activé sur IN8 (PA3).

## 3. Structure logicielle

Le programme fonctionne en simulant du multithreading. Chaque milliseconde, une interruption générée par le Timer 2 qui appelle successivement l'une des **4 fonctions du projet**, de manière cyclique.

### Les tâches sont les suivantes :

- **taskReadBtn**  
   Lit l’état du bouton utilisateur et met à jour la variable globale `motor_state`.

- **taskBlinkLed**  
   Gère l’affichage du message en morse via la LED, en fonction de `morse_btn_state`.

- **taskReadLed**  
   Lit la tension aux bornes du phototransistor via l’ADC. Si la tension dépasse un seuil, cela indique que le bon mot est affiché sur le codex.

-  **taskControlMotor**  
   Modifie le signal PWM selon l’état de l’énigme (`motor_state`), avec un délai d’une seconde pour éviter les activations accidentelles.

## 4. Détails de l’implémentation

### `main.c`

Le fichier "main" n'est utilisé uniquement pour initialiser les différent composants du projet : 

- Activation des interruptions (Timer 2)
- Lancement du signal PWM (Timer 1)

### `stm32l4xx_it.c`

Ce fichier comprend le cœur des fonctions du projet, exécuté à chaque interruption du Timer 2.

#### `taskReadBtn()`

Fonction simple mais essentielle :  
Elle lit l’état du bouton poussoir et met à jour la variable `morse_btn_state`. Cette variable déclenche ou interrompt la séquence Morse dans les autres fonctions.

#### `taskBlinkLed()`

Cette fonction affiche un mot en morse via une LED.  
- Des variables statiques gèrent le caractère affiché ainsi que sa durée d'exposition.
- À chaque appel, si le bouton a été pressé (`morse_btn_state == 1`), la LED clignote selon le code Morse définie dans une constante.
- En fin de message, les variables sont réinitialisées.

#### `taskReadLed()`

Fonction de lecture analogique :
- Lance une conversion ADC et attend la fin.
- Convertit la valeur lue (sur 12 bits, 0–4095) en tension via un produit en croix : $tension = \frac{100.Valeur~de~l'adc}{2^{12}}$. Ce qui permet d'avoir une valeur comprise entre 0 et 100 en sortie. On aurait pu multiplié par 5 pour obtenir la tensions aux bornes du photo-transistor en sortie, le choix de multiplié par 100 est purement arbitraire.
- Si la tension dépasse un seuil défini (ici 90), la variable globale `motor_state` est mise à 1 (sinon 0), indiquant que le codex a été résolu.

#### `taskControlMotor()`

Cette fonction gère le moteur via la PWM :
- Si `motor_state` passe à 1, on lance un compte à rebours d'une seconde avant de modifier le signal PWM.
- Cela évite que l’utilisateur n’active le moteur par hasard lors de la rotation du codex.
- Le PWM est modifié via `__HAL_TIM_SET_COMPARE()` en fonction de l’état validé.

Le rôle des moteurs dans l'énigme étant de simplement ouvrir ou fermé le prix. Ces derniers ne se situeront qu'au deux extrêmes possibles (0 ou 180°). C'est pourquoi, avec `__HAL_TIM_SET_COMPARE()`, nous paramétrons le signal pour qu'il soit sur un des deux extrêmes, c'est-à-dire une largeur de crénau de 10ms ou de 20ms.

NB : A savoir que lorsque l'on flash le code sur le microprocesseur, il faut ensuite couper l'alimentation de la carte avant de la remettre pour que le code prenne effet.
