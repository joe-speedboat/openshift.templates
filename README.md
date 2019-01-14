# openshift.templates
Some Templates I created for Openshift
## xfce-vnc template
For exam preparation and exploring all the functions of openshift, I created this template.   
Below you can find my notes about to create a template from existing deployment (see below)   
```
oc new-project project-name
oc new-app --name=app-name --docker-image=docker.io/christian773/xfce-vnc:latest VNC_PW=vnc-pw
oc expose svc/app-name --hostname=appurl.app.office.bitbull.ch --port=6901

oc export dc,is,svc,route --as-template=xfce-vnc > xfce-vnc.yml
cp xfce-vnc.yml xfce-vnc.yml.orig

sed -i '/namespace: /d' xfce-vnc.yml
sed -i 's@: docker-registry.default.svc.*@: docker.io/christian773/xfce-vnc:latest@' xfce-vnc.yml
sed -i 's@docker.io/christian773.*@docker.io/christian773/xfce-vnc:latest@' xfce-vnc.yml
sed -i 's/appurl.app.office.bitbull.ch/${APPLICATION_DOMAIN}/g' xfce-vnc.yml
sed -i 's/app-name/${NAME}/g' xfce-vnc.yml
sed -i 's/vnc-pw/${VNC_PW}/g' xfce-vnc.yml
echo 'parameters:
- description: The name assigned to all of the frontend objects defined in this template.
  displayName: Name
  name: NAME
  required: true
  value: xfce-vnc
- description: The exposed hostname that will route to the html5 web service, if left blank a value will be defaulted.
  displayName: Application Hostname
  name: APPLICATION_DOMAIN
- description: Password for VNC Login
  displayName: VNC Password
  name: VNC_PW
' >> xfce-vnc.yml

oc create -f xfce-vnc.yml
# oc create -f xfce-vnc.yml -n openshift
```

## xfce-vnc classroom
It came from the need to setup fast and cheap course environment.   
Below you can find my notes of how to create a BYOD classroom in 2 minutes:   
```
PROJECT=classroom
NR=5           # number of desktops to create
PASSWORD=xxx   # vnc password prefix
DOMAIN=app.domain.com
HOST=xfce
oc new-project $PROJECT
> $PROJECT-inventory.txt
seq -w $NR | while read NR
do
   URL=$HOST$NR.$DOMAIN
   oc new-app --name=$HOST$NR --docker-image=docker.io/christian773/xfce-vnc:latest VNC_PW=$PASSWORD$NR
   oc volume dc/$HOST$NR --add --name=$PROJECT -t pvc --overwrite \
   --claim-size=5G --claim-mode=ReadWriteMany --mount-path=/headless/Desktop/data --claim-name=$PROJECT
   oc expose svc/$HOST$NR --hostname=$URL --port=6901
   echo "https://$URL      Passwort: $PASSWORD$NR" >> $PROJECT-inventory.txt
done
cat $PROJECT-inventory.txt
```
