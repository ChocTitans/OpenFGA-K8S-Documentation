### **OpenFGA deployment in K8s**


## Requirements

- EBS CSI DRIVER `Will be added in CICD` [@OfficialDocumentation](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)

---


### **Kustomization**

We're using kustomization to deploy all the ressources, in the main root there's `kustomization.yaml` that specifies every deployment/service/pvc folder and also using it for secrets/configmap.


**Why too many kustomization file**

We have a file in the root called kustomization.yaml and it specifies the folder that has file manifests.
Let's say we have 5 folder and each of them has X file of manifests, to apply all of them we'll have to `cd` to each folder and use `kustomize build . | kubectl apply -f -` so instead of this, we'll create a folder in the root that calls all the kustomizations in each folder and we won't need to `cd` to all of them. we'll just execute it in the root and all the manifests will be applied

---

Add them to your repository's secrets : 

- Go to your repository's settings.

- Under the settings, navigate to the 'Secrets' section.

- Click on 'New repository secret' or 'Add a secret environment' button.

- For the name, enter `POSTGRES_DB` and the other variables ( `POSTGRES_PASSWORD`, `POSTGRES_USER`).

- Input the new master key value in the 'Value' field.

These secrets and configmap has to be defined in a workflow `Github Actions` : 

```
      - name: Create secrets file
        run: |
          cd ./k8s
          echo "POSTGRES_DB=${{ secrets.POSTGRES_DB }}" > secret.env
          echo "POSTGRES_PASSWORD=${{ secrets.POSTGRES_PASSWORD }}" >> secret.env
          echo "POSTGRES_USER=${{ secrets.POSTGRES_USER }}" >> secret.env
          echo "DATABASE_URL=postgresql://${{ secrets.POSTGRES_USER }}:${{ secrets.POSTGRES_PASSWORD }}@app-openfga-postgres:5432/${{ secrets.POSTGRES_DB }}" >> secret.env
          
      - name: Create configMap file
        run: |
          cd ./k8s
          echo "OPENFGA_PLAYGROUND_ENABLED=${{ vars.OPENFGA_PLAYGROUND_ENABLED }}" > configmap.env



```

in `DATABASE_URL`, @`name` has to match to postgres service's name and since we have `namePrefix: app-` in kustomization.yaml then it'll be `app-openfga-postgres` (by default)


It will create a secret/configmap based on these variables and can be modified only in Github repo's secret variables.


---

### **OpenFGA**

OpenFGA uses 2 DATASTORE_ENGINE, `memory` and `postgres`.

- `memory` : The memory storage engine is the default, but it is not persistent (data is lost between server restarts).
- `postgres` / `mysql` : The Postgres/mysql storage engine requires a Postgres instance that the OpenFGA server can reach. (needs postgres/mysql installed and `OPENFGA_DATASTORE_URI` defined)


Can be modified in :

File : `./openfga/deployment.yaml`
```
    - name: OPENFGA_DATASTORE_ENGINE
        value: "postgres"
```

For the URI can be modified in your github's repository secret.

File : `./openfga/deployment.yaml`
```
    - name: OPENFGA_DATASTORE_URI
        valueFrom:
        secretKeyRef:
            name: openfga-secret
            key: DATABASE_URL
```


### **Migrate**

We have a folder called migrate, it has `job.yaml` and `sa.yaml`, both of them are needed to migrate the database into OpenFGA, `sa.yaml` is responsible to start migrate's pod, without it we won't be able to store any data and we'll get an error :

```
{"code":"internal_error","message":"Internal Server Error"}

```

If migrate's pod is running but still getting this error, it means you forgot to setup an OPENFGA_DATASTORE_URI or a wrong URI (check your repository's secret) :

file : `./migrate/job.yaml`
```
    - name: OPENFGA_DATASTORE_URI
    valueFrom:
        secretKeyRef:
        name: openfga-secret
        key: DATABASE_URL
```

### **Postgres**

These are defined in postgres folder.

Same goes for `POSTGRES_PASSWORD`,`POSTGRES_USER`

File : `./postgres/deployment.yaml`
```
    - name: POSTGRES_DB
        valueFrom:
        secretKeyRef:
            name: openfga-secret
            key: POSTGRES_DB
```

---

Once everything is setup, To apply all the resources, run the following command:

`kustomize build . | kubectl apply -f - `

If you want to delete all the resources:

`kustomize build . | kubectl delete -f - `


### **Test**

Run this command to execute the openFGA service :

```
kubectl port-forward -n default service/app-openfga 8080:8080
```

Run this command to see if everything is working and navigate to `http://localhost:8080/stores` :

```
curl -X POST 'localhost:8080/stores' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "openfga-demo"
}'
```

If you get this message it means it works as intended.

```
{"id":"01H7068VWQH7VGC8TPYYVN0DP5","name":"openfga-demo","created_at":"2023-08-04T11:52:34.969669Z","updated_at":"2023-08-04T11:52:34.969669Z"}

```

### **Ingress**


Request your domain in AWS ACM. Once the request is approved and the certificate is issued, copy the Certificate ARN in `./openfga/ingress.yaml`

File : `./openfga/ingress.yaml` (Line 10)
```
    alb.ingress.kubernetes.io/certificate-arn: <yourcertificate> # Certificate ARN in ACM (for your domain)
```

Next, change the domain to your desired domain:

File : `ingress.yaml` (Line 16)
```
    - host: "your.domain.com"
```


