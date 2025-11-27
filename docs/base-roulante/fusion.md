# Fusion de capteurs

---

- *Fichiers : `tinyefk.h`, `fusion.h` et `fusion.c`*

> [!CAUTION] Contenu non validé
> Pour l'instant, cette explication a été générée par une IA. Après une relecture rapide, la théorie semble cohérente et bien présentée, mais **je ne peux garantir l'exactitude des calculs ou des mathématiques utilisées**.
>
> Pour approfondir le sujet, je vous recommande vivement de consulter les **ressources humaines** disponibles à la fin du chapitre. Elles offrent des explications fiables et détaillées.

## 1. Principe de la fusion de capteurs

L'objectif est de combiner les données de l'**odométrie** (précise à court terme mais sujette à la dérive) et de l'**IMU** (mesure de l'accélération et de la vitesse angulaire, mais bruyante) pour obtenir une estimation plus robuste de la position et de l'orientation du robot.

## 2. Modèle du système

### 2.1. État du robot

L'état du robot est défini par :

$$
\mathbf{x} = \begin{bmatrix} x \\ y \\ \theta \\ v_x \\ v_y \\ \omega \end{bmatrix}
$$

- $(x, y)$ : Position du robot.
- $\theta$ : Orientation.
- $(v_x, v_y)$ : Vitesse linéaire.
- $\omega$ : Vitesse angulaire.

### 2.2. Modèle dynamique

Le modèle prédit l'évolution de l'état en fonction des entrées (vitesses des roues) :

$$
\dot{\mathbf{x}} = \begin{bmatrix}
v_x \cos(\theta) - v_y \sin(\theta) \\
v_x \sin(\theta) + v_y \cos(\theta) \\
\omega \\
a_x \\
a_y \\
\alpha
\end{bmatrix}
$$

- $(a_x, a_y)$ : Accélérations linéaires (mesurées par l'IMU).
- $\alpha$ : Accélération angulaire (mesurée par l'IMU).

## 3. Modèle du filtre de Kalman

### 3.1. Équations du filtre de Kalman

Le filtre de Kalman se compose de deux étapes :

**Prédiction** :

$$
\hat{\mathbf{x}}_k^- = \mathbf{F}_k \hat{\mathbf{x}}_{k-1} + \mathbf{B}_k \mathbf{u}_k
$$

$$
\mathbf{P}_k^- = \mathbf{F}_k \mathbf{P}_{k-1} \mathbf{F}_k^T + \mathbf{Q}_k
$$

- $\mathbf{F}_k$ : Matrice de transition d'état.
- $\mathbf{B}_k$ : Matrice de contrôle.
- $\mathbf{u}_k$ : Entrées de contrôle (vitesses des roues).
- $\mathbf{Q}_k$ : Matrice de covariance du bruit de processus.

**Mise à jour** :

$$
\mathbf{K}_k = \mathbf{P}_k^- \mathbf{H}_k^T (\mathbf{H}_k \mathbf{P}_k^- \mathbf{H}_k^T + \mathbf{R}_k)^{-1}
$$

$$
\hat{\mathbf{x}}_k = \hat{\mathbf{x}}_k^- + \mathbf{K}_k (\mathbf{z}_k - \mathbf{H}_k \hat{\mathbf{x}}_k^-)
$$

$$
\mathbf{P}_k = (\mathbf{I} - \mathbf{K}_k \mathbf{H}_k) \mathbf{P}_k^-
$$

- $\mathbf{H}_k$ : Matrice d'observation.
- $\mathbf{z}_k$ : Mesures des capteurs (odométrie + IMU).
- $\mathbf{R}_k$ : Matrice de covariance du bruit de mesure.
- $\mathbf{K}_k$ : Gain de Kalman.

## 4. Intégration de l'odométrie et de l'IMU

### 4.1. Mesures de l'odométrie

Les données d'odométrie fournissent une estimation de $(v_x, v_y, \omega)$ à partir des vitesses des roues.

### 4.2. Mesures de l'IMU

L'IMU fournit :

- **Accélérations linéaires** $(a_x, a_y)$ dans le repère du robot.
- **Vitesse angulaire** $\omega$ (gyroscope).

### 4.3. Fusion des données

Les mesures de l'IMU sont utilisées pour corriger la dérive de l'odométrie, tandis que l'odométrie fournit une estimation précise des déplacements à court terme.

## 5. Implémentation du filtre de Kalman

### 5.1. Initialisation

```python
import numpy as np

# État initial (position, orientation, vitesses)
x = np.array([0.0, 0.0, 0.0, 0.0, 0.0, 0.0])

# Matrice de covariance initiale
P = np.eye(6)

# Matrices de transition et de contrôle
F = np.eye(6)
F[0, 3] = dt * np.cos(x[2])  # dt : période d'échantillonnage
F[1, 3] = dt * np.sin(x[2])
F[2, 5] = dt
F[3, 3] = 1.0
F[4, 4] = 1.0

B = np.zeros((6, 3))  # Matrice de contrôle pour les entrées (v_x, v_y, ω)

# Matrices de covariance des bruits
Q = np.eye(6) * 0.1  # Bruit de processus
R_imu = np.eye(3) * 0.5  # Bruit de mesure IMU (accélérations + gyro)
R_odo = np.eye(3) * 0.1  # Bruit de mesure odométrie (v_x, v_y, ω)
```

### 5.2. Étape de prédiction

```python
def predict(x, P, u, F, B, Q, dt):
    # u : entrées de contrôle [v_x, v_y, ω]
    x[0] += u[0] * dt * np.cos(x[2]) - u[1] * dt * np.sin(x[2])
    x[1] += u[0] * dt * np.sin(x[2]) + u[1] * dt * np.cos(x[2])
    x[2] += u[2] * dt
    x[3:6] = u  # Mise à jour des vitesses

    # Mise à jour de la matrice de transition F
    F[0, 3] = dt * np.cos(x[2])
    F[1, 3] = dt * np.sin(x[2])

    # Prédiction de la covariance
    P = F @ P @ F.T + Q
    return x, P
```

### 5.3. Étape de mise à jour

```python
def update(x, P, z_imu, z_odo, H_imu, H_odo, R_imu, R_odo):
    # z_imu : mesures IMU [a_x, a_y, ω]
    # z_odo : mesures odométrie [v_x, v_y, ω]

    # Fusion des mesures (exemple : moyenne pondérée)
    z = np.concatenate((z_odo[:2], z_imu[2:]))  # Utiliser v_x, v_y de l'odométrie et ω de l'IMU
    H = np.vstack((H_odo[:2], H_imu[2:]))  # Matrice d'observation combinée
    R = np.diag(np.concatenate((np.diag(R_odo)[:2], np.diag(R_imu)[2:])))

    # Gain de Kalman
    S = H @ P @ H.T + R
    K = P @ H.T @ np.linalg.inv(S)

    # Mise à jour de l'état et de la covariance
    y = z - H @ x
    x = x + K @ y
    P = (np.eye(6) - K @ H) @ P
    return x, P
```

## 6. Exemple complet d'intégration

```python
# Boucle principale
while True:
    # Lecture des capteurs
    v_x, v_y, omega_odo = read_odometry()
    a_x, a_y, omega_imu = read_imu()

    # Entrées de contrôle
    u = np.array([v_x, v_y, omega_odo])

    # Prédiction
    x, P = predict(x, P, u, F, B, Q, dt)

    # Mise à jour
    z_imu = np.array([a_x, a_y, omega_imu])
    z_odo = np.array([v_x, v_y, omega_odo])
    x, P = update(x, P, z_imu, z_odo, H_imu, H_odo, R_imu, R_odo)

    # Affichage ou utilisation de l'état estimé
    print(f"Position estimée : x={x[0]:.2f}, y={x[1]:.2f}, θ={x[2]:.2f}")
```

## 7. Optimisations et bonnes pratiques

### 7.1. Réglage des matrices de covariance

- **Q** : Ajustez en fonction du bruit attendu sur le modèle dynamique.
- **R** : Ajustez en fonction de la confiance dans chaque capteur (ex : l'IMU est souvent plus bruyante que l'odométrie pour les vitesses linéaires).

### 7.2. Filtrage complémentaire

- Utilisez un **filtre complémentaire** pour fusionner les angles de l'IMU (gyroscope + accéléromètre) et éviter la dérive du gyroscope.

### 7.3. Validation expérimentale

- Testez le filtre dans un environnement simulé (ex : Gazebo) avant de le déployer sur le robot réel.
- Visualisez les estimations avec `matplotlib` pour détecter les divergences.

## 8. Outils et bibliothèques recommandés

- **Python** : `numpy`, `scipy`, et `filterpy` (pour une implémentation facile du filtre de Kalman).
- **ROS** : Utilisez le package `robot_localization` pour une fusion de capteurs avancée.
- **Simulateurs** : Gazebo ou PyBullet pour valider le filtre dans un environnement virtuel.

## Ressources et Références

- [Lightweight C/C++ Extended Kalman Filter](https://github.com/simondlevy/TinyEKF/tree/master)
- [Interactive Tutorial for Non-Experts](https://simondlevy.github.io/ekf-tutorial/)
