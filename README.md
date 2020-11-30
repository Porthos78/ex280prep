# 1. Labels #
##
oc label node node1 env=dev 

oc label node node2 env=prod

oc label node node1 env=test --overwrite

oc get nodes --show-labels

oc label node node1 env-

# 2. Node Selector #
##
oc patch deployment/myapp --patch '{"spec":{"template":{"spec":{"nodeSelector":{"env":"dev"}}}}}'

oc adm new-project demo --node-selector "env=dev"

oc annotate namespace demo openshift.io/node-selector "env=dev" --overwrite

oc patch namespace demo --patch '{"metadata":{"annotations":{"openshift.io/node-selector": "env=dev"}}}'

# 3. Taints #
##
oc adm taint nodes node1 dedicated=foo:NoSchedule -o json --dry-run=client | jq .spec.taints
```
[
  {
    "effect": "NoSchedule",
    "key": "dedicated",
    "value": "foo"
  }
]
```

oc describe node node1 |grep -A2 Taint
```
Taints:      dedicated=foo:NoSchedule
             test=foo:NoSchedule
             
```
oc adm taint node node1 dedicated-

oc adm taint node node1 test-

# 4. OAuth #
sudo yum install httpd-tools

htpasswd -c -b /tmp/htpass user1 password1

htpasswd -b /tmp/htpass user2 password2

oc get oauth cluster -o yaml > /tmp/oauth.yaml

add httpasswd to oauth.yaml
```
spec:
  identityProviders:
  - name: localusers
    type: HTPasswd
    mappingMethod: claim
    htpasswd:
      fileData:
        name: /tmp/htpass
```
oc replace -f /tmp/oauth.yaml

oc create secret generic htpass-secret --from-file htpasswd=/tmp/htpass -n openshift-config

oc get po -w -n openshift-authentication

htpasswd -D /tmp/htpass user2 # *delete user*

oc set data secret/htpass-secret --from-file htpasswd=/tmp/htpass -n openshift-config

oc delete user user2

oc delete identity localusers:user1

oc delete user --all
oc delete identity --all

# 5. Users, Groups, and Authentication #
oc __adm__ policy add-cluster-role-to-user cluster-admin user-name    #__cluster role__

oc policy add-role-to-user role-name user-name -n project-name        #__namespace level role__

oc get clusterrolebindings -o wide |grep -E "NAME|self-provisioners"

oc describe clusterrolebindings self-provisioners

oc describe clusterrole self-provisioner

Remove self-provisioner role from system such that authenticated users can't create projects<br/>
&nbsp;&nbsp;&nbsp;&nbsp;oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth

Restore self-provisioners back to cluster as original<br/>
&nbsp;&nbsp;&nbsp;&nbsp;oc adm policy add-cluster-role-to-group --rolebinding-name self-provisioners self-provisioner system:authenticated:oauth

oc adm group new dev-users dev1 dev2

oc adm group new qa-users qa1 qa2

oc policy add-role-to-group __edit__ dev-users -n namespace

oc policy add-role-to-group __view__ qa-users -n namespace
oc policy add-role-to-user admin user1 -n namespace
oc 
