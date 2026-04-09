MultiTool
App Android modulare con sistema di moduli integrati.
Struttura repository
MultiTool/
├── app/                        <- Host MultiTool (launcher, lista moduli, log globale)
├── module-ffmpeg/              <- Modulo FFmpeg (libreria Android)
│   └── libs/                   <- POSIZIONARE QUI ffmpeg-kit-full.aar (non incluso nel repo)
├── .github/workflows/          <- Build automatica via GitHub Actions
├── settings.gradle             <- include ':app' e ':module-ffmpeg' + flatDir
└── README.md
Setup iniziale OBBLIGATORIO
Prima di compilare devi posizionare il file .aar:
Copia ffmpeg-kit-full.aar nella cartella module-ffmpeg/libs/
Fai commit e push su GitHub
GitHub Actions compila automaticamente e produce l'APK scaricabile da Actions > Artifacts
Regole critiche per i moduli (imparate in produzione)
1. Nomi layout con prefisso
Il layout principale del modulo NON si può chiamare activity_main.xml.
Causerebbe un conflitto silenzioso con il layout dell'host e crash a runtime.
Usare sempre il prefisso del modulo: ffmpeg_activity_main.xml, imagetool_activity_main.xml, ecc.
Stessa regola per tutti i file in res/ (drawable, values, layout).
2. flatDir solo in settings.gradle
Il flatDir per i file .aar locali va dichiarato SOLO in settings.gradle:
flatDir { dirs 'module-ffmpeg/libs' }
NON va dichiarato nel build.gradle del modulo. Causerebbe errore con PREFER_SETTINGS.
3. useLegacyPackaging in app/build.gradle
Le librerie native .so di ffmpeg-kit richiedono:
packaging { jniLibs { useLegacyPackaging = true } }
Va dichiarato sia in module-ffmpeg/build.gradle che in app/build.gradle.
4. Dipendenze transitive degli AAR locali
Gli AAR locali non portano con sé le loro dipendenze automaticamente.
ffmpeg-kit-full richiede questa dipendenza aggiunta manualmente:
implementation 'com.arthenica:smart-exception-java:0.2.1'
Ogni nuovo modulo che usa un AAR locale deve verificare le sue dipendenze transitive.
5. CheckAarMetadata disabilitato
Necessario in entrambi i build.gradle:
tasks.withType(com.android.build.gradle.internal.tasks.CheckAarMetadataTask).configureEach {
    enabled = false
}
Aggiungere un nuovo modulo
Creare cartella module-nomemodulo/ con struttura Android library
settings.gradle: aggiungere include ':module-nomemodulo'
app/build.gradle: aggiungere implementation project(':module-nomemodulo')
ModuleRegistry.kt: aggiungere ModuleInfo(icon, name, desc, "com.exe.nomemodulo.MainActivity")
Push su GitHub. Actions compila automaticamente.
Output APK
GitHub Actions > ultimo build > Artifacts > app-debug.zip
