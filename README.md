# 🧲 Équation d'Allen-Cahn — Simulation numérique par différences finies

> Résolution numérique de l'équation d'Allen-Cahn par schémas explicite et implicite, avec simulations 1D, 2D et ajout d'un terme de transport.

**Cours :** Projet EDP — ESILV 2024/2025  
**Auteurs :** Cléa Marin · Raphael Marques Araujo

---

## 📋 Table des matières

- [Contexte physique](#contexte-physique)
- [L'équation d'Allen-Cahn](#léquation-dallen-cahn)
- [Méthodes numériques](#méthodes-numériques)
- [Simulations implémentées](#simulations-implémentées)
- [Structure du dépôt](#structure-du-dépôt)
- [Installation](#installation)
- [Utilisation](#utilisation)
- [Résultats](#résultats)

---

## Contexte physique

L'équation d'Allen-Cahn est une équation aux dérivées partielles (EDP) décrivant la **séparation de phases dans les alliages binaires**. Elle est utilisée en physique des matériaux pour modéliser la répartition spatiale des structures d'un alliage après un processus de chauffage ou de refroidissement rapide.

Le champ $u(x,t)$ représente la **phase du matériau** : ses valeurs $+1$ et $-1$ correspondent aux deux phases stables de l'alliage, et l'interface entre les deux évolue selon la dynamique de l'équation.

---

## L'équation d'Allen-Cahn

L'équation est posée sur un domaine $\Omega \times (0, T)$ avec conditions de Dirichlet homogènes :

$$\frac{\partial u}{\partial t} - \varepsilon \Delta u + f(u) = 0$$

$$u(x, 0) = u_0(x), \quad u(x,t)\big|_{\partial\Omega} = 0$$

avec le terme non linéaire :

$$f(u) = F'(u), \quad F(u) = \frac{1}{4}(u^2 - 1)^2 \implies f(u) = u^3 - u$$

| Variable | Description |
|----------|-------------|
| $u(x,t)$ | Champ de phase du matériau |
| $\varepsilon$ | Paramètre positif dépendant du matériau |
| $\Omega$ | Domaine spatial (ouvert) |

---

## Méthodes numériques

### Discrétisation

Le domaine spatio-temporel est discrétisé avec $N$ points en espace (pas $\Delta x$) et des pas de temps $\Delta t$. On note $u_i^n \approx u(x_i, t_n)$.

On pose $\alpha = \dfrac{\varepsilon \Delta t}{(\Delta x)^2}$.

### Schéma explicite

La dérivée temporelle et le Laplacien sont évalués au temps $t^n$ :

$$u_i^{n+1} = u_i^n(1 - 2\alpha) + \alpha u_{i+1}^n + \alpha u_{i-1}^n - \Delta t\left((u_i^n)^3 - u_i^n\right)$$

Sous forme matricielle :

$$u^{n+1} = A_{\text{exp}}\, u^n - \Delta t\left((u^n)^{\circ 3} - u^n\right)$$

où $A_{\text{exp}}$ est une matrice tridiagonale de diagonale $(1-2\alpha)$ et sur-diagonales $\alpha$.

**Consistance :** ordre 1 en temps, ordre 2 en espace — $\mathcal{O}(\Delta t) + \mathcal{O}(\Delta x^2)$

**Condition CFL (stabilité) :**

$$\frac{\varepsilon \Delta t}{\Delta x^2} \leq \frac{1}{2}$$

### Schéma implicite

Le Laplacien est évalué au temps $t^{n+1}$, ce qui conduit à un système non linéaire :

$$A_{\text{imp}}\, u^{n+1} + \Delta t\left((u^{n+1})^{\circ 3} - u^{n+1}\right) = u^n$$

où $A_{\text{imp}}$ est tridiagonale de diagonale $(1+2\alpha)$ et sur-diagonales $-\alpha$.

Ce système non linéaire est résolu à chaque pas de temps par la **méthode de Newton**, inconditionnellement stable.

**Consistance :** ordre 1 en temps, ordre 2 en espace — $\mathcal{O}(\Delta t) + \mathcal{O}(\Delta x^2)$

---

## Simulations implémentées

| # | Simulation | Schéma | Dimension |
|---|-----------|--------|-----------|
| 1 | Solution manufacturée (validation) | Explicite | 1D |
| 2 | Simulation générale | Implicite | 1D |
| 3 | Champ de phase sur domaine carré | Explicite | 2D |
| 4 | Transition de phase à front vertical | Explicite | 1D & 2D |
| 5 | Allen-Cahn avec terme de transport | Explicite | 1D & 2D |

### Transition de phase (cas connu)

Le champ $u$ est initialisé avec deux phases distinctes :
- $u = +1$ à gauche
- $u = -1$ à droite
- Interface lisse à $u = 0$ au centre

Cela représente deux matériaux en contact. La simulation montre le lissage progressif de l'interface par diffusion.

### Terme de transport

L'équation est enrichie d'un terme d'advection :

$$\frac{\partial u}{\partial t} = \varepsilon \Delta u - f(u) - v \frac{\partial u}{\partial x}$$

avec une vitesse de transport $v = 0.3$ m/s. Ce terme modélise un déplacement du champ, par exemple sous l'effet d'un flux thermique ou mécanique.

---

## Résultats

### Schéma explicite vs implicite (1D)

| Propriété | Explicite | Implicite |
|-----------|-----------|-----------|
| Stabilité | Conditionnelle (CFL) | Inconditionnelle |
| Convergence erreur | Rapide | Plus lente mais niveau plus faible |
| Résolution | Directe | Itérative (Newton) |
| Ordre temps / espace | 1 / 2 | 1 / 2 |

- Le schéma **explicite** converge rapidement mais est soumis à la contrainte CFL $\varepsilon \Delta t / \Delta x^2 \leq 1/2$.
- Le schéma **implicite** est inconditionnellement stable ; l'erreur inter-itération converge vers des niveaux très faibles, et la solution $u(x,t)$ décroît rapidement entre $t=0.5$ et $t \approx 1$ avant de se stabiliser vers zéro.

### Simulations 2D

Les simulations 2D confirment la cohérence avec les résultats 1D : la phase évolue progressivement dans le domaine central avant de respecter les conditions aux bords. La séparation nette entre les deux phases est conservée, et l'interface évolue de manière isotrope.

### Terme de transport

L'ajout du terme $-v \partial u / \partial x$ produit un déplacement visible du champ vers la droite, superposé à la diffusion, ce qui correspond bien à un écoulement dirigé à vitesse constante.

---

## Références

- Allen, S. M. & Cahn, J. W., *A microscopic theory for antiphase boundary motion*, Acta Metallurgica (1979)
- Finite difference methods for PDEs — cours ESILV EDP 2024/2025

---

## Licence

Projet académique — ESILV Projet EDP 2024/2025.
