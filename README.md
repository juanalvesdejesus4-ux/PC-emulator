import os, textwrap, zipfile

root = "/mnt/data/PC_Virtual_Android"
os.makedirs(root, exist_ok=True)

files = {
"settings.gradle": """pluginManagement { repositories { google(); mavenCentral(); gradlePluginPortal() } }
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories { google(); mavenCentral() }
}
rootProject.name = "PCVirtualAndroid"
include(":app")
""",
"build.gradle": """plugins {
    id 'com.android.application' version '8.7.3' apply false
    id 'org.jetbrains.kotlin.android' version '2.0.21' apply false
}
""",
"app/build.gradle": """plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
}
android {
    namespace 'com.example.pcvirtual'
    compileSdk 35
    defaultConfig {
        applicationId 'com.example.pcvirtual'
        minSdk 23
        targetSdk 35
        versionCode 1
        versionName '1.0'
    }
}
""",
"app/src/main/AndroidManifest.xml": """<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET"/>
    <application android:theme="@style/AppTheme" android:label="PC Virtual">
        <activity android:name=".MainActivity" android:screenOrientation="landscape"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
""",
"app/src/main/res/values/styles.xml": """<resources>
    <style name="AppTheme" parent="android:style/Theme.Material.Light.NoActionBar">
        <item name="android:fontFamily">sans</item>
        <item name="android:windowFullscreen">true</item>
        <item name="android:colorAccent">#0078D7</item>
    </style>
</resources>
""",
"app/src/main/java/com/example/pcvirtual/MainActivity.kt": """package com.example.pcvirtual

import android.app.*
import android.os.Bundle
import android.graphics.Color
import android.content.Intent
import android.net.Uri
import android.view.Gravity
import android.widget.*

class MainActivity : Activity() {
    private lateinit var desktop: LinearLayout
    private lateinit var taskbar: LinearLayout

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        buildDesktop()
    }

    private fun buildDesktop() {
        val root = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            setBackgroundColor(Color.rgb(30, 70, 120))
        }

        desktop = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            gravity = Gravity.TOP
            setPadding(25, 25, 25, 10)
        }
        root.addView(desktop, LinearLayout.LayoutParams(-1, 0, 1f))

        taskbar = LinearLayout(this).apply {
            gravity = Gravity.CENTER_VERTICAL
            setPadding(8, 4, 8, 4)
            setBackgroundColor(Color.rgb(25, 25, 25))
        }
        val start = Button(this).apply {
            text = "⊞  Iniciar"
            setOnClickListener { showStartMenu() }
        }
        taskbar.addView(start, LinearLayout.LayoutParams(120, 55))
        root.addView(taskbar, LinearLayout.LayoutParams(-1, 60))

        addDesktopIcon("📁\\nArquivos") { fileManager() }
        addDesktopIcon("🌐\\nNavegador") { browser() }
        addDesktopIcon("📝\\nBloco de Notas") { notepad() }
        addDesktopIcon("🧮\\nCalculadora") { calculator() }
        setContentView(root)
    }

    private fun addDesktopIcon(label: String, action: () -> Unit) {
        val b = Button(this).apply {
            text = label
            textSize = 15f
            setTextColor(Color.WHITE)
            setBackgroundColor(Color.TRANSPARENT)
            setOnClickListener { action() }
        }
        desktop.addView(b, LinearLayout.LayoutParams(180, 80))
    }

    private fun showStartMenu() {
        val box = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            setPadding(20, 15, 20, 15)
            setBackgroundColor(Color.DKGRAY)
        }
        val dialog = Dialog(this)
        listOf("🌐 Navegador", "📁 Arquivos", "📝 Bloco de Notas",
            "🧮 Calculadora", "⚙️ Configurações").forEach { name ->
            val b = Button(this).apply {
                text = name
                setOnClickListener {
                    dialog.dismiss()
                    when {
                        name.contains("Navegador") -> browser()
                        name.contains("Arquivos") -> fileManager()
                        name.contains("Notas") -> notepad()
                        name.contains("Calculadora") -> calculator()
                        else -> settings()
                    }
                }
            }
            box.addView(b)
        }
        dialog.setContentView(box)
        dialog.window?.setBackgroundDrawableResource(android.R.color.transparent)
        dialog.show()
    }

    private fun browser() {
        val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://www.google.com"))
        startActivity(intent)
    }

    private fun fileManager() {
        startActivity(Intent(Intent.ACTION_OPEN_DOCUMENT).apply {
            type = "*/*"
            addCategory(Intent.CATEGORY_OPENABLE)
        })
    }

    private fun notepad() {
        val edit = EditText(this).apply {
            hint = "Digite seu texto..."
            gravity = Gravity.TOP
            setPadding(20,20,20,20)
        }
        AlertDialog.Builder(this).setTitle("Bloco de Notas")
            .setView(edit)
            .setPositiveButton("Fechar", null).show()
    }

    private fun calculator() {
        val input = EditText(this).apply { hint = "Ex.: 12 + 5"; inputType = 2 }
        AlertDialog.Builder(this).setTitle("Calculadora")
            .setView(input)
            .setPositiveButton("OK") { _, _ ->
                Toast.makeText(this, "Use a calculadora do Android para cálculos completos.", Toast.LENGTH_LONG).show()
            }.show()
    }

    private fun settings() {
        AlertDialog.Builder(this).setTitle("Configurações")
            .setMessage("PC Virtual\\nVersão 1.0\\n\\nInterface inspirada em desktop.")
            .setPositiveButton("OK", null).show()
    }
}
""",
"README.md": """# PC Virtual Android

Projeto Android de uma interface de desktop inspirada em Windows 10.

## Como compilar
1. Abra a pasta no Android Studio.
2. Aguarde o Gradle baixar as dependências.
3. Vá em Build > Build APK(s).
4. O APK será gerado em `app/build/outputs/apk/debug/`.

## Importante
Este projeto NÃO contém o Windows 10 nem o Google Chrome original.
O navegador abre o navegador padrão do Android. Para usar Chrome, instale o Chrome separadamente no aparelho.

O projeto é uma interface/simulador de desktop, não uma máquina virtual completa do Windows.
"""
}

for path, content in files.items():
    full = os.path.join(root, path)
    os.makedirs(os.path.dirname(full), exist_ok=True)
    with open(full, "w", encoding="utf-8") as f:
        f.write(content)

zip_path = "/mnt/data/PC_Virtual_Android_Projeto.zip"
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for base, _, names in os.walk(root):
        for name in names:
            full = os.path.join(base, name)
            z.write(full, os.path.relpath(full, root))

zip_path
