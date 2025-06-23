# Networks of organisations

**Research Question**: Do specific universities or other institutions increase the likelihood of winning a Nobel Prize? Have these institutions produced multiple Nobel laureate over time?

## Eigenvector vs betweenness, with number of persons

![Distribution of continets](../../notebooks_jupyter/wikidata_exploration/images/images_network/revenus_eigenvector_20210526.jpg "Distribution of the continets")

* Columbia University and the University of Freiburg both score near the top of betweenness with a lower eigenvector. They act as a bridge telling us that many laureates have passed through them in career transitions, collaborations, or joint appointments.
* University of Cambridge, Ludwig-Maximilians-Universität München, and Humboldt-Universität zu Berlin rank at the far right. They have many laureates (large bubbles) and are also linked to other major institutions which shows that they’re influential and in the center of the global network.

## Nobel laureate network 

![Distribution of continets](../../notebooks_jupyter/wikidata_exploration/images/images_network/communite.png "Distribution of the continets")

The network was created with 13 components (is_connected: False)  where there is one outstanding component with 104 nodes with 7 communities. Each colour is one of those communities that represents a set of universities that have many more ties among themselves than out to the rest of the world. 

## Results

Being connected to a top elite, research universities greatly boosts the likelihood of winning a Nobel Prize. Institutions like the University of Cambridge, Harvard, LMU Munich and Humboldt-Berlin are situated at the very center of the laureate network (high eigenvector) and they’ve produced many Nobel winners over decades.
