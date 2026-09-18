# Uitvoerbare CI-samples

De broncode staat in [woozer/ci-samples](https://github.com/woozer/ci-samples). Haal dit project afzonderlijk op met:

```sh
git clone https://github.com/woozer/ci-samples.git
cd ci-samples
```

De pipelines draaien in de [lokale GitLab-demo](https://github.com/woozer/ci-components/blob/main/installation.md). De onderstaande `localhost`-links werken na installatie. Voor zelfstandig bouwen en testen: [ontwikkelhandleiding](docs/development.md).

Open [New pipeline](http://localhost:8929/root/ci-samples/-/pipelines/new), selecteer `main`, kies `sample` en klik op **New pipeline**. Met `all` voer je alle negentien voorbeelden op de beschermde `main` uit. Gebruik bij `library_ref` standaard de uitgebrachte componentversie `2.0.0`, kies een andere volledige versie of geef een volledige commit-SHA op om een kandidaatwijziging te testen. De [versieafspraken](https://github.com/woozer/ci-components/blob/main/docs/component-versions.md) gelden voor alle opgenomen bestanden.

Kies `java-service` om de standaardpipeline uit [ci-pipelines](https://github.com/woozer/ci-pipelines) uit te voeren met backend, UI en scans in modus `validate`. `pipeline_ref` selecteert daarvoor de pipelineversie (standaard `1.1.0`) of kandidaat-SHA; deze pipeline beheert haar eigen vaste moduleversies. Deployment en applicatierelease horen bij de volledige validatie in **hello-world**. `all` blijft de negentien componentvoorbeelden uitvoeren.

Kies voor losse bouwblokken een van de vier pipelinevoorbeelden of een van de vijftien losse modules, zoals `module-sonar`, `module-dependency-check` of `module-npm-test`. Sonar en de drie releasevoorbeelden vereisen de beschermde `main`; `all` in een merge request voert de overige vijftien voorbeelden uit.

De pipelines gebruiken rechtstreeks de [voorbeeld-YAML voor afnemers](https://github.com/woozer/ci-components/tree/main/examples/samples). De validatie voegt alleen lokale testinstellingen, controles en cleanup toe. Modulewijzigingen starten dit project automatisch met de gewijzigde component-SHA.

Deze repository bevat de volledige testapplicatie en de eigen sampleconfiguratie. De applicatie is afgeleid van [hello-world](https://github.com/woozer/hello-world), maar wordt hier afzonderlijk beheerd. Werk de testapplicatie bij via een merge request. De [installer](https://github.com/woozer/ci-components/blob/main/installation.md) vult het lokale GitLab-project met de vastgelegde commit van deze repository.

De testkopie gebruikt daarnaast dezelfde Tomcat-beveiligingsupdate naar 11.0.25 als `hello-world`. `environment/cluster/test.yaml` levert de routing voor het losse Helm-voorbeeld. `deploy.yml` controleert de gekozen profielen wanneer het voorbeeld `deployment-select` een childpipeline start.

Build- en testartifacts blijven in GitLab. Sample-images en -charts gaan naar `docker-local/root-ci-samples/`. Deployments gebruiken namespace `ci-samples`, API-poort 8180 en UI-poort 8190. De parenttrigger houdt een lock vast tijdens deployment, tests en cleanup. Cleanup verwijdert de tijdelijke Helm-releases. Na annulering kan het nodig zijn de cleanup-job handmatig opnieuw te starten.

## Waar staan de pipelines?

De `.gitlab-ci.yml` in dit project bevat het keuzeformulier voor beide bibliotheken. Voor componentvoorbeelden neemt hij `tests/samples/launcher.yml` uit **ci-components** op; die start de gekozen voorbeelden als childpipelines. Voor `java-service` neemt hij de sampleconfiguratie uit **ci-pipelines** op, die de publieke cataloguscomponent gebruikt. De bibliotheken hebben elk hun eigen versie en validatiepipeline.

- [`examples/modules/`](https://github.com/woozer/ci-components/tree/main/examples/modules) bevat een uitvoerbaar voorbeeld per actieve module.
- [`examples/samples/`](https://github.com/woozer/ci-components/tree/main/examples/samples) bevat de pipelines die afnemers kunnen overnemen.
- [`tests/samples/`](https://github.com/woozer/ci-components/tree/main/tests/samples) bevat onze launcher, testinstellingen, outputcontroles en cleanup.

De launcher staat onder `tests/` omdat hij de validatie organiseert. Deze mappenindeling is onze keuze; GitLab schrijft die niet voor. `include:inputs` geeft instellingen door aan opgenomen YAML. Keuzes op **New pipeline** komen uit `spec:inputs` van de hoofdconfiguratie.

Lees de [modulehandleiding](https://github.com/woozer/ci-components/blob/main/docs/modules.md) en [handleiding voor de samples](https://github.com/woozer/ci-components/blob/main/examples/samples/README.md).

## Publicatie- en releasetests

De tests publiceren echte Maven-packages, images en charts met unieke versies. De releasevoorbeelden reserveren tags en maken GitLab-releases uitsluitend in dit project. Ze gebruiken een eigen deploy key zonder pushrechten op `main`. De artifacts blijven beschikbaar voor controle; versies en tags worden niet hergebruikt. De Sonar-sample schrijft naar het afzonderlijke SonarQube-project `ci-samples`.

Het volledige overzicht staat in de [handleiding voor losse modulevoorbeelden](https://github.com/woozer/ci-components/blob/main/examples/modules/README.md).

De scannerresultaten blijven beschikbaar als artifacts. Dependency-Check levert ook JUnit voor **Tests** en het MR-testoverzicht, plus een reactie met aantallen en bevindingen. De centrale Sonar-helper haalt gate en meetwaarden via de API op na analyse van main; het resultaat komt bij de commit en, indien van toepassing, de bijbehorende gemergede MR. Zie [rapportage en beperkingen](https://github.com/woozer/ci-components/blob/main/docs/scanners.md).
