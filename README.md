# Quantum Transport Clustering

The package `quantum_transport_clustering` contains three major class objects:

- `GraphMethods`: construct undirected graphs, and compute and encapsulate their graph Laplacians
- `SpectralClustering`: perform correct spectral clustering on undirected graph Laplacians
- `QuantumTransportClustering`: perform quantum transport clustering on undirected graph Laplacians

Usage example

```python
import quantum_transport_clustering as qtc
graph_ = qtc.GraphMethods(data)
...
spec = qtc.SpectralClustering(n_clusters=3, norm_method='row')
...
shot = qtc.QuantumTransportClustering(n_clusters=3, Hamiltonian=Lap_)
```

## Graph Methods

```python
quantum_transport_clustering.GraphMethods(data_, graph_embedded=True, edt_tau = None, eps_quant = None, normed=True, compute_lap = True)
```

The Class `GraphMethods` is able to

- Generate Gaussian RBF adjacency matrix using Euclidean distances of the data distribution
- Compute Graph Lapalcian (symmetrically normalized by default)
- Store the raw data as well as adjacency matrix and graph Laplacian

| Parameters       |                                          |
| ---------------- | :--------------------------------------- |
| `data_`          | If `graph_embedded = True`, `data_` is a numpy array of shape ($n_\text{feature}$, $m_\text{sample}$), or  $m_\text{sample}$ points in $\mathbb R^{n_\text{feature}}$. If `graph_embedded = False`, `data_` is a numpy array of shape ( $m_\text{sample}$, $m_\text{sample}$) representing the adjacency of a graph with  $m_\text{sample}$ nodes. |
| `graph_embedded` | `bool`, optional. If `True`, assume the graph is embedded in a Euclidean space. If `False` , assume the input data set is an adjacency matrix not *a priori* embedded in a Euclidean space. |
| `edt_tau`        | `int`, $\tau > 0$, optional. If specified, it is the number of iterations of effective dissimilarity transformation (EDT). Neglected if `graph_embedded = False`. |
| `eps_quant`      | `float`, in range $0<\varepsilon<100$, optional. The the quantile of distance distribution. If not specified, $\varepsilon = 1$. Neglected if `graph_embedded = False`. |
| `normed`         | `bool`, optional. If `False`, graph Laplacian is $L = D - A$ where $D$ is degree diagonal matrix, and $A$ the adjacency matrix. If `True`, graph Laplacian will be normalized $H = D^{-\frac12} L D^{-\frac12}$. |
| `compute_lap`    | `bool`, optional. If `True`, graph Laplacian will be computed upon initialization. |

| Returns |                                          |
| ------- | :--------------------------------------- |
| `Lap_`  | numpy array of shape ( $m_\text{sample}$, $m_\text{sample}$). The graph Laplacian matrix $L$ or $H$. |

Example:

```python
graph_ = qtc.GraphMethods(data)
laplacian_matrix_ = graph_.Lap_
```



## Spectral Clustering

```python
quantum_transport_clustering.SpectralClustering(n_clusters, norm_method='row', isExact=True)
```
Perform correct spectral clustering on undirected graph Laplacians.


| Parameters    |                                          |
| ------------- | ---------------------------------------- |
| `n_clusters`  | `int`, $n_\text{cluster} > 0$ , the number of clusters. |
| `norm_method` | `None`, `"row"`, or `"deg"`. If `None`, the spectral embedding is not normalized. If `"row"`, the spectral embedding is $L^2$-normalized by row where each row represent a node. If `"deg"`, the spectral embedding is normalized by degree vector. |
| `isExact`     | `bool`. If `True`, exact eigenvalues and eigenvectors will be computed. If `False`, first  (small) $n_\text{cluster}$ eigenvalues and eigenvectors will be computed. |

| Methods     |                                          |
| ----------- | ---------------------------------------- |
| `fit(Lap_)` | `Lap_` is the symmetric graph Laplacian. First, the eigenvalues and eigenstates are computed. Next, perform spectral embedding and $k$-means. |

| Returns   |                                          |
| --------- | ---------------------------------------- |
| `labels_` | An integer-valued numpy array of shape ($m_\text{sample}$). The class labels associated with each node. |

Example:

```python
spec = qtc.SpectralClustering(n_clusters=3, norm_method='row')
spec.fit(laplacian_matrix)
spec_labels_ = spec.labels_
```



## Quantum Transport Clustering

```python
quantum_transport_clustering.QuantumTransportClustering(n_clusters, Hamiltonian, s=1.0, isExact=True, n_eigs = None)
```

Perform quantum transport clustering on undirected graph Laplacians.

| Parameters    |                                          |
| ------------- | ---------------------------------------- |
| `n_clusters`  | `int` , $n_\text{cluster} > 0$ , the number of clusters. |
| `Hamiltonian` | numpy array of shape ($m_\text{sample}$, $m_\text{sample}$). The symmetric graph Laplacian matrix $H$. |
| `s`           | `float`, $\tilde s>0$ , optional. The actual $s$-parameter of Laplace transform will be $s = \tilde s \times (E_{n_\text{cluster} - 1} - E_0) / (n_\text{cluster} - 1)$, where $E_n$ are eigenvalues of $H$. |
| `isExact`     | `bool`, optional. If `True`, exact eigenvalues and eigenvectors of $H$ will be computed. If `False`, first `n_eigs` low energy states will be computed approximated. |
| `n_eigs`      | `int`, $n_\text{eigs} > 0$, optional. If `n_eigs` not specified and `isExact = False`, $n_\text{eigs} = 10\times n_\text{cluster}$. If `n_eigs` is specified and `isExact = True`, then first $n_\text{eigs}$ low exact energy state will be used to perform quantum transport clustering. The latter case can be used to speed up the clustering processes. |

| Methods      |                                          |
| ------------ | ---------------------------------------- |
| `Grind()`    | `Grind(s=None, grind='medium', method_='diff', init_nodes_=None)`  Option `grind` can be `"coarse"`, `"medium"`, `"fine"`, `"micro"`, or `custom`. Option `method` can be `diff` or `kmeans` corresponding to direct difference and $k$-means methods. If `grind="custom"`, then `init_nodes_`  is the custom python `list` of initialization nodes. Method `Grind()` produce the array `Omg_`  is the matrix $\Omega$ which contains the raw class labels. |
| `Espresso()` | Perform "direct extraction method" on $\Omega$. This method creates attribute `labels_` as the predicted class labels. |
| `Coldbrew()` | Compute "consensus matrix" $C$ based on $\Omega$. This method creates attribute `consensus_matrix_`. |

| Returns             |                                          |
| ------------------- | ---------------------------------------- |
| `Omg_`              | An integer-valued numpy array of shape ($m_\text{sample}$, $m_\text{initialization}$). The raw class labels of $m_\text{samples}$ from quantum transport from $m_\text{initialization}$ nodes. |
| `labels_`           | An integer-valued numpy array of shape ($m_\text{sample}$). The final prediction by `Espresso()`. |
| `consensus_matrix_` | A float-valued numpy array of shape ($m_\text{sample}$, $m_\text{sample}$). The consensus matrix computed by `Coldbrew()`. |

Example:

```python
shot = qtc.QuantumTransportClustering(n_clusters=3, Hamiltonian=Lap_)
shot.Grind()
shot.Espresso()
class_labels_ = shot.labels_
shot.Coldbrew()
C_matrix_ = shot.consensus_matrix_
```

