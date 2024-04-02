### Job
```
apiVersion: batch/v1
kind: Job
metadata:
  name: <runner name>
  namespace: <namespace>
spec:
  template:
    metadata:
      labels:
        accountUuid: <accountUuid>
        runnerUuid: <runnerUuid>
    spec:
      containers:
        - name: bitbucket-runner-02
          image: "docker-public.packages.atlassian.com/sox/atlassian/bitbucket-pipelines-runner"
          env:
            - name: ACCOUNT_UUID
              value: "{<accoundUuid>}"
            - name: RUNNER_UUID
              value: "{<runnerUuid>}"
            - name: REPOSITORY_UUID
              value: "{<repositoryUuid>}"
            - name: RUNTIME_PREREQUISITES_ENABLED
              value: "true"
            - name: WORKING_DIRECTORY
              value: "/tmp"
            - name: OAUTH_CLIENT_ID
              valueFrom:
                secretKeyRef:
                  name: <runner name>-oauth-credentials
                  key: oauthClientId
            -  name: OAUTH_CLIENT_SECRET
               valueFrom:
                 secretKeyRef:
                   name: <runner name>-oauth-credentials
                   key: oauthClientSecret
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: docker-containers
              mountPath: /var/lib/docker/containers
              readOnly: true
            - name: var-run
              mountPath: /var/run
        - name: docker-in-docker
          image: docker:20.10.7-dind
          securityContext:
            privileged: true
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: docker-containers
              mountPath: /var/lib/docker/containers
            - name: var-run
              mountPath: /var/run
      restartPolicy: OnFailure
      volumes:
        - name: tmp
        - name: docker-containers
        - name: var-run
  backoffLimit: 6
  completions: 1
  parallelism: 1
```

### Secret
```
apiVersion: v1
kind: Secret
metadata:
  name: <name of runner>-oath-credentials
  namespace: <namespace>
  labels:
    accountUuid: <accountUuid>
    runnerUuid: <runnerUuid>
data:
  oauthClientId: "<base64 encoded oauth client id>"
  oauthClientSecret: "<base64 encoded oauth client secret>"
```
  
