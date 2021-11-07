# Jupyter

- [https://www.anaconda.com/](https://www.anaconda.com/)
- [https://www.anaconda.com/products/individual](https://www.anaconda.com/products/individual)
- Start jupyter lab on commandline
  ```
  cd code/jupyternotebooks/
  jupyter lab
  ```
  
- Local web notebook [http://localhost:8888/lab](http://localhost:8888/lab)
---
- [https://jupyter.org/](https://jupyter.org/)
- [jupyterlab - ReadTheDocs](https://jupyterlab.readthedocs.io/en/latest/)
- [JupyterLab Install](https://jupyterlab.readthedocs.io/en/stable/getting_started/installation.html)
- [youtube Demo - CM 9.1 - Jupyter Notebook: In-depth Overview](https://www.youtube.com/watch?v=JWiRLx9M2R4)
---
- [https://mybinder.org/](https://mybinder.org/)
- [tbd]()
- [tbd]()
- [FULL-FEATURED BRIGHT CLUSTER MANAGEMENT SOFTWARE FOR CLUSTERS OF UP TO 8 NODES](https://www.brightcomputing.com/easy8)
- [tbd]()

## Setup [Jupyter on k8s](https://medium.com/analytics-vidhya/deploying-standalone-jupyterlab-on-kubernetes-for-early-stage-startups-7a1468fae289)
1. Create jlab namespace 
```
kubectl create namespace jlab
```

- Jupyterlab install
```
kubectl apply -f jupyterlab-pvc.yaml
kubectl apply -f jupyterlab-deployment.yaml
kubectl apply -f jupyterlab-service.yaml

# change permission of home folder in pvc 
# since newly created pvc don't give permission 
# to jupyter user (jovian) to create notebook files
chmod 777 /home/jovyan

# you must perform above step with `sleep infinity` in command
# or can have init-container that does that for you
```

- [github raw - jupyterlab-pvc.yaml](https://raw.githubusercontent.com/2cld/rackn.2cld.net/main/docs/jupyter/jupyterlab-pvc.yaml)
- [github raw - jupyterlab-deployment.yaml](https://raw.githubusercontent.com/2cld/rackn.2cld.net/main/docs/jupyter/jupyterlab-deployment.yaml)
- [github raw - jupyterlab-service.yaml](https://raw.githubusercontent.com/2cld/rackn.2cld.net/main/docs/jupyter/jupyterlab-service.yaml)

- Download from github using [wget and/or curl](https://gist.github.com/jwebcat/5122366)
```
wget --no-check-certificate --content-disposition https://raw.githubusercontent.com/2cld/rackn.2cld.net/main/docs/jupyter/jupyterlab-deployment.yaml
# --no-check-cerftificate was necessary for me to have wget not puke about https
curl -LJO https://raw.githubusercontent.com/2cld/rackn.2cld.net/main/docs/jupyter/jupyterlab-deployment.yaml

```

2. Get external-ip of your deployed service
```
kubectl get svc -n jlab
```

3. Navigate to the external-ip in your browser to access JupyterLab

