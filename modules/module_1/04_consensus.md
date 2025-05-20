---
id: consensus
sidebar_label: Consensus
---

# 1.4 Consensus — AleoBFT

Aleo utilise AleoBFT, un consensus basé sur un graphe orienté acyclique (DAG), inspiré de Narwhal et Bullshark, avec des comités dynamiques de validateurs.

## Structure
- **Narwhal** : Mempool DAG pour la diffusion efficace des transactions.
- **Bullshark** : Détermine l'ordre total des transactions à partir du DAG.

### Narwhal
- Transactions = sommets du DAG
- Références = arêtes
- Les validateurs forment des certificats d'après les endorsements reçus

### Bullshark
- Sélectionne un leader à chaque round pair
- Si l'ancre reçoit assez de votes, elle est committée
- Garantit la cohérence et la finalité

## Avantages
- **Haute performance** (pas de leader unique)
- **Moins de surcharge** (certificats digest)
- **Résilience** (pas de view-change)

AleoBFT est formellement vérifié, scalable et adapté aux applications blockchain confidentielles.

![AleoBFT Diagram 1](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252Fp5g3uj2L31JEIneYBGMc%252Fimage.png%3Falt%3Dmedia%26token%3D2af50154-fac3-41ef-9019-546f647263f3&width=768&dpr=4&quality=100&sign=d73a7d57&sv=2)

![AleoBFT Diagram 2](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252Fx58dKMWyM3B2kOtQSX05%252Fimage.png%3Falt%3Dmedia%26token%3D4838cc68-9dc3-4fe4-9b5f-0582d78da08e&width=768&dpr=4&quality=100&sign=ffa77bcb&sv=2)

![AleoBFT Diagram 3](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FUSvGj4CEGpQbu5y6Zyx9%252Fimage.png%3Falt%3Dmedia%26token%3Df2722b17-2067-418f-9e9e-09c577dd760d&width=768&dpr=4&quality=100&sign=7bfa8367&sv=2)

![AleoBFT Diagram 4](https://zlearn.gitbook.io/~gitbook/image?url=https%3A%2F%2F2329510431-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252FvhcF8MdVyA1W0ElVe7v6%252Fuploads%252FqOwnqF8RDWLOuSBPKYmk%252Fimage.png%3Falt%3Dmedia%26token%3D6a08e0a6-1149-45e7-8389-9a8b716d8c63&width=768&dpr=4&quality=100&sign=3e58afd9&sv=2) 