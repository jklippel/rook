# DO NOT MERGE

This file is a todo list for the implementation of integration tests for the rook/ceph/swift with keystone functionality.

## Was wollen wir testen?

### Swift 

Das gibts alles noch nicht, bin mir aber nicht sicher, wie wir das außerhalb des Clusters testen wollen.
Dann bräuchte es noch einen Ingress-Weg zum Keystone bzw. einen von extern erreichbaren Service.

- (implementiert) Container anlegen
- (implementiert) Datei in Container ablegen
- (implementiert) Datei aus Container laden
- (implementiert) Datei in Container löschen
- (implementiert) Container löschen
- (implementiert) unberechtigter Zugriff (jan)
- berechtigte Zugriffe (Read-only, Write-only(?), Admin?) (jan)

### S3 (silvio)

Wie testen wir das? (mit welchem S3-Client?, kann man das mit openstack-cli machen?)

- Bucket anlegen
- Bucket verwenden
- Bucket löschen
- unberechtigter Zugriff
- berechtigte Zugriffe (Read, Write, Admin)

## S3 Credentials in Keystone anlegen anstelle des ObjectStoreUsers aus den normalen Tests (silvio)
-> Access keys aus Keystone in das jetzige Secret legen(?)

## CephObjektStoreUser + Subuser (silvio)

- Was machen wir mit der Funktionalität?
- (done) Trennen wir den MR/PR/Branch auf? --> ja
- (done) Testen wir diese Funktionalität auch? --> in einem weiteren PR

## Spezifikation lesen und schauen ob wir alles abgedeckt haben (jan/silvio)

ich (jan) finde es sieht ganz gut aus, allerdings sind da natürlich die Subuser Sachen beschrieben. Aber ich denke
dass man nicht alles in einem PR implementieren muss.

## nach erstem rudimentären mal den PR aufmachen und die Community einbinden

## (low-prio) Nebenschauplatz: tests/scripts/generate-tls-config.sh kubectl version --short schlägt fehl, denn es ist nicht mehr aktuell

## (low-prio) Ceph-Swift-Dokumentation anschauen und verbessern

## (low-prio) einen Blick auf Coverage werfen

## (low-prio) Schauen, warum das ceph_manifest.go Dinge verwenden möchte, die nicht im CRD sind

Scheint ein Entwurf gewesen zu sein. Ich (Jan) würde es erstmal rauslassen. Zumal mir das Konzept nicht klar ist:
Einzelne Buckets prüfen? Oder einen bestimmten? Immer wieder abfragen in dem Interval ob der noch da ist?

Der hier angegebene 'type:' ist deprecated laut Design-Dokument
The currently ignored `gateway.type` option is deprecated and from now on
explicitly ignored by rook.


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

## (low-prio) Brauchen wir ein neueres Keystone als Yoga? (Jan)

Das letzte Yaook-Image ist für Yoga, aber man könnte da ruhig auch ein paar neuere bauen.
(Ist mir beim Doku-Lesen aufgefallen :)
https://gitlab.com/yaook/images/keystone/-/merge_requests/100


## (done) COSI-Fehler / wo kommen die her? (jan)
failed to reconcile CephObjectStore "keystoneauth-ns/default". failed to create object store deployments: failed to get COSI user "cosi": Get "http://rook-ceph-rgw-default.keystoneauth-ns.svc:80/admin/user?format=json&uid=cosi": dial tcp 10.106.142.173:80: connect: connection refused

Upstream tests nochmal anschauen
-> Taucht in den upstream logs nicht auf

## Wie testen wir das? (Aus dem Design-Dokument)

> It must be possible to serve S3 and Swift for the same object store
> pool.