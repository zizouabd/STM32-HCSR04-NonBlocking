# STM32 HC-SR04 : Mesure Ultrason Non-Bloquante via USB-C

Ce projet implémente une lecture de distance continue (25 Hz) avec un capteur à ultrasons HC-SR04 sur une carte STM32F411CEU6 (Black Pill). Les données sont transmises à un PC via le port USB-C intégré (Virtual COM Port).

## 🎯 Objectifs et Justification des Choix Techniques

L'objectif principal de ce projet est de s'affranchir des limitations des codes classiques pour Arduino, en tirant pleinement parti de l'architecture matérielle du STM32.

### 1. Architecture 100% Hardware (Non-Bloquante)
Les approches classiques utilisent des fonctions `Delay()` et des boucles `while` bloquantes qui monopolisent le processeur pendant la durée du trajet de l'onde sonore.
* **Le choix :** Toute la gestion du capteur est déléguée aux Timers matériels (Hardware-driven).
* **TIM3 (PWM sur PA6) :** Génère automatiquement l'impulsion de déclenchement (TRIG) de 10 µs à une fréquence précise de 25 Hz.
* **TIM2 (Input Capture sur PA0) :** Mesure la largeur de l'impulsion de retour (ECHO) avec une résolution d'une microseconde (1 MHz).
* **Bénéfice :** Le CPU est libre à 99% pour exécuter d'autres tâches. Une interruption ne se déclenche que lorsque la mesure est terminée.

### 2. Optimisation de la Mémoire Flash (Gestion des Floats)
La fonction `sprintf` standard avec le support activé des nombres à virgule flottante (`%f`) ajoute environ 15 Ko de surcoût à la mémoire Flash du microcontrôleur.
* **Le choix :** La distance calculée en `float` est scindée mathématiquement en deux variables entières (partie entière et première décimale). 
* **Bénéfice :** Cela permet d'utiliser un formatage simple `%d.%d`, gardant le binaire extrêmement léger tout en conservant une précision d'affichage de 0.1 cm.

### 3. Horloge Système et USB-CDC
* L'horloge système (SYSCLK) est poussée à **96 MHz** via le PLL pour une réactivité maximale.
* Le diviseur `PLLQ` est strictement réglé pour fournir **48 MHz** au périphérique USB_OTG_FS.
* **Bénéfice :** La carte communique directement avec le PC via le même câble USB-C utilisé pour l'alimentation, éliminant le besoin d'un convertisseur UART-vers-USB (FTDI) externe.

### 4. Sécurité Matérielle (Abaissement de tension)
Le capteur HC-SR04 fonctionne avec une logique 5V, tandis que l'STM32 fonctionne en 3.3V. Bien que de nombreuses broches de l'STM32F411 soient tolérantes au 5V.
* **Le choix :** Un pont diviseur de tension (résistances de 1 kΩ et 2 kΩ) a été placé sur la broche de retour ECHO.
* **Bénéfice :** Le signal entrant est abaissé à environ 3.3V, respectant les standards électriques natifs du MCU et évitant tout stress à long terme sur la broche PA0.

## 🛠️ Matériel et Câblage

| HC-SR04 | Broche STM32F411 | Remarque |
| :--- | :--- | :--- |
| **VCC** | 5V | Alimenté directement par le port USB |
| **GND** | GND | Masse commune |
| **TRIG** | PA6 | Connecté directement (Sortie TIM3_CH1) |
| **ECHO** | PA0 | Connecté via un pont diviseur de tension (Entrée TIM2_CH1) |

## 🚀 Utilisation

1. Compilez le projet sous STM32CubeIDE.
2. Flashez le firmware (fichier `.elf` ou `.hex`) en mode DFU (boutons BOOT0 + NRST) via STM32CubeProgrammer.
3. Appuyez sur le bouton `NRST` pour lancer le programme.
4. Ouvrez un terminal série (ex: STM32CubeIDE Terminal, Arduino Monitor, PuTTY) et connectez-vous au port COM de la carte.
5. Réglez la vitesse sur **115200 baud**. Les distances s'affichent automatiquement à 25 Hz.
