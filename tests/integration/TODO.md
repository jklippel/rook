# DO NOT MERGE

This file is a todo list for the implementation of integration tests for the rook/ceph/swift with keystone functionality.

## Keystone-Deployment hinzufügen

### (done) Via Yaook-Operator oder abgestript via Manifest-Dateien

ja, Tendenz: abgestrippte Manifest-Dateien (siehe nächster Abschnitt)

### (done) abgestrippte Manifest-Dateien

Ich hab jetzt eine Keystone-Deployment-Version ohne Yaook aber mit TLS.

Das findet sich im rook-minikube-keystone repository unter resources/keystone-only.

Da gibt es zwei Skripte:

- provision-keystone.sh
- deprovision-keystone.sh

Das legt Keystone ohne Yaook im Rook-Test/Dev-Minikube Cluster an. Inklusive Cert-Manager-Installation und ca-issuer.

Das kann man mit `kubectl apply -n keystone -f osc.yaml` ebenfalls deployen.
Credentials und die notwendige Variable zur CA-Datei sind bereits im Environment des Deployments enthalten.

Eine Shell im Pod starten:

```sh
kubectl exec -n keystone -ti deployment/osc -- bash
```
Dann kann man, wenn der keystone-api Pod "Ready" ist, mit dem OpenStack Client auf das Keystone zugreifen.

```sh
openstack endpoint list
```

und auch User anlegen:

```sh
openstack user create alice --password 4l1c3
```
** alice heißt jetzt rook-user sonst ändert sich nix

### (done) TLS muss drin bleiben

Ja, ist noch drin. Ich habs direkt in den Apache eingebaut (siehe Datei `apache2.conf`).
Das man den Cert-Manager braucht find ich grad noch etwas unschön.

### (done) SQLite reicht aus

Ja, keystone verwendet jetzt sqlite. (siehe `keystone.conf`)
Die Datenbank wird von einem Init-Container befüllt.
Ein weiterer Init-Container legt die ersten Endpoints an.

### (done) keystone-manage bootstrap in init-Container?
Ist integriert. Das ist der, der die ersten Init-Container anlegt.

### (done) Präferenz: Wenns klein geht mittels Manifest-Dateien

Ja ich denke es ist recht übersichtlich geworden. Die Cert-Manager Installation find ich noch nicht so toll,
aber zum Starten reichts auf jeden Fall.

### Integration der Manifest-Dateien in rook-Integration-Test

### (done) Keystone-Only-Deployment in `ceph_base_keystone.go` übertragen.

Alle Resourcen sind in `ceph_base_keystone_test.go` integriert.

Der Test läuft auch durch.

### (done) Das keystone-api-Deployment wird angelegt und dann wieder gelöscht.

## Schauen, warum das cephfs getestet wird

## OpenStack-Library (gophercloud) oder swift-Client verwenden?

https://github.com/ncw/swift
oder
https://pkg.go.dev/github.com/gophercloud/gophercloud@v1.8.0/openstack/objectstorage/v1/containers

## Was wollen wir testen?

### Swift

Das gibts alles noch nicht, bin mir aber nicht sicher, wie wir das außerhalb des Clusters testen wollen.
Dann bräuchte es noch einen Ingress-Weg zum Keystone bzw. einen von extern erreichbaren Service.

- Container anlegen
- Datei in Container ablegen
- Datei aus Container laden
- Datei in Container löschen
- Container löschen
- unberechtigter Zugriff
- berechtigte Zugriffe (Read, Write, Admin)
- Smoke-Test? (many requests to keystone)

### S3

- Bucket anlegen
- Bucket verwenden
- Bucket löschen
- unberechtigter Zugriff
- berechtigte Zugriffe (Read, Write, Admin)

## CephObjektStoreUser + Subuser

- Was machen wir mit der Funktionalität?
- Trennen wir den MR/PR/Branch auf?
- Testen wir diese Funktionalität auch?

## Spezifikation lesen und schauen ob wir alles abgedeckt haben

## Ceph-Swift-Dokumentation anschauen und verbessern

## nach erstem rudimentären mal den PR aufmachen und die Community einbinden

## einen Blick auf Coverage werfen

## Schauen, warum das ceph_manifest.go Dinge verwenden möchte, die nicht im CRD sind

```
diff --git a/tests/framework/installer/ceph_manifests.go b/tests/framework/installer/ceph_manifests.go
index 5cba689e2..21aa32ee8 100644
--- a/tests/framework/installer/ceph_manifests.go
+++ b/tests/framework/installer/ceph_manifests.go
@@ -498,14 +498,9 @@ spec:
   {{ end }}
   gateway:
     resources: null
-    type: s3
     {{ if .TLS }}securePort: {{ .Port }}{{ else }}port: {{ .Port }}{{ end }}
     instances: {{ .ReplicaCount }}
-    {{ if .TLS }}sslCertificateRef: {{ .Name }}{{ end }}
-  healthCheck:
-    bucket:
-      disabled: false
-      interval: {{ if .TLS }}10s{{ else }}5s{{ end }}`
+    {{ if .TLS }}sslCertificateRef: {{ .Name }}{{ end }}`

        return renderTemplate(tmpl, spec)
 }

```

### GetKeystoneUserSecret anpassen

In object.go die Funktion GetKeystoneUserSecret anpassen/variabler machen

### Todos in ceph_object_test.go anschauen (Parameter swiftAndKeystone sind immer false)

## Nebenschauplatz: tests/scripts/generate-tls-config.sh kubectl version --short schlägt fehl, denn es ist nicht mehr aktuell