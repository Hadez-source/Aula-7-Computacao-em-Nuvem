Aula 7 computação em nuvem


Código Parte 1: 

root@controlplane:~$ kubectl get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   11d   v1.37.1
node01         Ready    <none>          11d   v1.37.1
root@controlplane:~$ kubectl create namespace aula7
namespace/aula7 created
root@controlplane:~$ kubectl get namespace aula7
NAME    STATUS   AGE
aula7   Active   11s
root@controlplane:~$ kubectl -n aula7 get secret credencial-demo
Error from server (NotFound): secrets "credencial-demo" not found
root@controlplane:~$ kubectl -n aula7 create secret generic credencial-demo \
  --from-literal=senha=exemplo-123
secret/credencial-demo created
root@controlplane:~$ 
kubectl -n aula7 get secret credencial-demo
NAME              TYPE     DATA   AGE
credencial-demo   Opaque   1      13s
root@controlplane:~$ cat > pod-secret.yaml <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: leitor-secret
  namespace: aula7
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ['sh', '-c', 'sleep 3600']
    volumeMounts:
    - name: credencial
      mountPath: /segredos
      readOnly: true
  volumes:
  - name: credencial
    secret:
      secretName: credencial-demo
EOF
root@controlplane:~$ kubectl apply -f pod-secret.yaml
pod/leitor-secret created
root@controlplane:~$ kubectl -n aula7 wait --for=condition=Ready pod/leitor-secret --timeout=90s
pod/leitor-secret condition met
root@controlplane:~$ kubectl -n aula7 exec leitor-secret -- ls /segredos
senha
root@controlplane:~$ kubectl -n aula7 exec leitor-secret -- cat /segredos/senha
exemplo-123root@controlplane:~$ kubectl -n aula7 create serviceaccount leitor
serviceaccount/leitor created
root@controlplane:~$ kubectl -n aula7 get serviceaccount leitor
NAME     AGE
leitor   19s
root@controlplane:~$ kubectl -n aula7 create role consulta-pods \
  --verb=get,list --resource=pods
role.rbac.authorization.k8s.io/consulta-pods created
root@controlplane:~$ kubectl -n aula7 describe role consulta-pods
Name:         consulta-pods
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [get list]
root@controlplane:~$ kubectl -n aula7 create rolebinding vinculo-leitor \
  --role=consulta-pods --serviceaccount=aula7:leitor
rolebinding.rbac.authorization.k8s.io/vinculo-leitor created
root@controlplane:~$ kubectl -n aula7 describe rolebinding vinculo-leitor
Name:         vinculo-leitor
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  consulta-pods
Subjects:
  Kind            Name    Namespace
  ----            ----    ---------
  ServiceAccount  leitor  aula7
root@controlplane:~$ kubectl auth can-i get pods -n aula7 \
  --as=system:serviceaccount:aula7:leitor
yes
root@controlplane:~$ kubectl auth can-i delete pods -n aula7 \
  --as=system:serviceaccount:aula7:leitor
no



Perguntas (analise de resultados):
1 O que muda ao usar Secret em vez de ConfigMap neste exemplo?
2 Qual objeto define get e list? Qual objeto entrega essas permissões à conta?
3  Por que delete pods retornou no?
4  Uma conta com essa Role poderia ler Secrets? Justifiquem.

1- O Secret é usado para guardar informações sensíveis, como senhas, enquanto o ConfigMap guarda configurações comuns
2- A Role define as permissões e o RoleBinding passa essas permissões para a conta
3- Porque a conta só tem permissão para consultar e listar Pods, não para excluir.
4- Uma conta com essa Role poderia ler Secrets? Justifiquem.

