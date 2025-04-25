# GanttView
Standalone HTML Gantt-Tool (dhtmlxGantt) zum Anzeigen, Bearbeiten, Speichern (.gantt) &amp; Exportieren (PDF/PNG/CSV). Optionales Supabase-Sharing (Teilen via Link oder Einbetten via iframe). GPL-2.0.

## Beschreibung

GanttView ist ein einfacher, clientseitiger Betrachter und Editor für Gantt-Diagramme, der in einer einzigen HTML-Datei enthalten ist. Er ermöglicht das lokale Laden, Bearbeiten und Speichern von Projekten im `.gantt`-Format (JSON) sowie den Export der Ansichten.

Optional können, nach Einrichtung eines eigenen Supabase-Backends, Gantt-Diagramme über einen permanenten Link geteilt oder als iFrame in andere Webseiten eingebettet werden.

Dieses Projekt wurde für interne Zwecke in einer öffentlichen Verwaltung (Stadt Linz, Österreich) entwickelt und wird als Open Source zur Verfügung gestellt.

## Features

* Freies Erstellen von Projekten (Tasks hinzufügen und bearbeiten).
* Laden von Projekten aus `.gantt`-Dateien (JSON-Format).
* Speichern von Projekten in `.gantt`-Dateien (JSON-Format).
* Visualisierung von Projekten als Gantt-Diagramm mittels [dhtmlxGantt](https://dhtmlx.com/docs/products/dhtmlxGantt/).
* Alternative Darstellung der Projektaufgaben in einer bearbeitbaren Tabellenansicht.
* Bearbeiten von Tasks via Lightbox (im Gantt-Diagramm) oder durch Inline-Bearbeitung (in der Tabelle).
* Unterstützung für Aufgabenabhängigkeiten (Verknüpfungen).
* Anpassbare Farben für Aufgabenbalken.
* Export der Gantt-Diagramm-Ansicht als PNG-Bild oder PDF-Dokument.
* Export der Tabellendaten als CSV-Datei.
* **Optional:** Teilen eines permanenten Links zum aktuellen Projektstand (erfordert Supabase-Setup).
* **Optional:** Generieren eines iFrame-Embed-Codes für die Gantt-Diagramm-Ansicht (erfordert Supabase-Setup).

## Lizenz

Dieses Projekt (der HTML/CSS/JavaScript-Code in dieser Datei) ist unter der **GNU General Public License v2.0 (GPL-2.0)** lizenziert. Der vollständige Lizenztext ist in der Datei `LICENSE` verfügbar, die diesem Projekt beiliegt.

**Wichtiger Hinweis zur dhtmlxGantt-Abhängigkeit:**

Dieses Tool basiert maßgeblich auf der **dhtmlxGantt-Bibliothek** ([https://dhtmlx.com/docs/products/dhtmlxGantt/](https://dhtmlx.com/docs/products/dhtmlxGantt/)), die über ein CDN geladen wird. dhtmlxGantt ist **nicht** Teil dieses Open-Source-Projekts und unterliegt eigenen, separaten Lizenzbedingungen (kommerziell oder GPLv2).

Laut der dhtmlx-Website (Stand April 2025): *"Open-source DHTMLX Version - Standard Edition
The open source versions of DHTMLX Gantt, Scheduler, and Suite (Standard Editions) are distributed under the GPL v2.0 license. If you have an open-source project licensed under a GPLv2-compatible license and do not need PRO features, you may use these products for free. These versions do not come with official technical support, but you can access assistance through the community forum"*

Dieses Projekt (GanttView Standalone) wird unter GPL-2.0 veröffentlicht, unter anderem um potenziell diese Bedingung für die kostenlose Nutzung der dhtmlxGantt *Standard Edition* zu erfüllen.

**Benutzer dieses Codes sind jedoch allein verantwortlich für:**

1.  Die Überprüfung, dass sie **nur Funktionen der Standard Edition** von dhtmlxGantt verwenden (siehe dhtmlx-Dokumentation).
2.  Die Sicherstellung, dass ihre spezifische Nutzung mit den Lizenzbedingungen von dhtmlxGantt (Kommerziell oder GPLv2) übereinstimmt.

## Abhängigkeiten

* **[dhtmlxGantt](https://dhtmlx.com/docs/products/dhtmlxGantt/):** (Lizenz: Kommerziell oder GPLv2 - Erfordert separate Nutzerverifizierung/Lizenzierung durch dhtmlx). Geladen via CDN.
* **[jsPDF](https://github.com/parallax/jsPDF):** (MIT Lizenz). Geladen via CDN.
* **[html2canvas](https://html2canvas.hertzen.com/):** (MIT Lizenz). Geladen via CDN.
* **[jspdf-autotable](https://github.com/simonbengtsson/jsPDF-AutoTable):** (MIT Lizenz). Geladen via CDN.
* **[Supabase Client (supabase-js)](https://github.com/supabase/supabase-js):** (MIT Lizenz). Geladen via CDN. Nur für optionale Teilen/Einbetten-Funktionen benötigt.
* **Google Fonts (Roboto, Roboto Mono):** (SIL OFL / Apache Lizenz). Geladen via CDN.

## Setup und Konfiguration

Dieses Tool kann vollständig lokal ohne zusätzliche Einrichtung für das Laden, Bearbeiten, Speichern von `.gantt`-Dateien und lokale Exporte (PNG, PDF, CSV) verwendet werden.

Um die **optionalen Funktionen 'Link teilen' und 'Einbetten'** zu aktivieren, müssen Sie ein (z.B. kostenloses) Supabase Backend-Projekt einrichten:

**1. Klonen oder Herunterladen:**
   Besorgen Sie sich die `ganttview.html` Datei (oder klonen Sie dieses Repository).

**2. Supabase-Projekt erstellen:**
   * Gehen Sie zu [https://supabase.com/](https://supabase.com/), registrieren Sie sich oder melden Sie sich an.
   * Erstellen Sie ein neues Projekt. Wählen Sie eine Region und vergeben Sie ein sicheres Datenbankpasswort (dieses wird für das Tool selbst nicht benötigt, aber bewahren Sie es sicher auf). 

**3. Supabase-Datenbank einrichten:**
   * Navigieren Sie im Dashboard Ihres Supabase-Projekts zum 'SQL Editor'.
   * Klicken Sie auf '+ New query'.
   * Führen Sie das folgende SQL-Skript aus. Es erstellt die notwendige Tabelle `gantt_shares` und aktiviert die empfohlenen Row Level Security (RLS) Policies für anonymes Teilen:

   ```sql
   -- Tabelle erstellen, um geteilte Gantt-Daten zu speichern
   CREATE TABLE public.gantt_shares (
     id uuid DEFAULT gen_random_uuid() NOT NULL PRIMARY KEY,
     created_at timestamp with time zone DEFAULT now() NOT NULL,
     -- Speichert das serialisierte Gantt-Objekt { data: [], links: [] }
     gantt_data jsonb NULL
   );

   -- Optional: Index für schnellere Suche nach ID hinzufügen
   CREATE INDEX gantt_shares_id_idx ON public.gantt_shares USING btree (id);

   -- WICHTIG: Row Level Security (RLS) für die Tabelle aktivieren
   ALTER TABLE public.gantt_shares ENABLE ROW LEVEL SECURITY;

   -- Policies erstellen:
   -- 1. Erlaube jedem (anon-Rolle), neue Zeilen einzufügen (notwendig zum Speichern für Teilen/Einbetten)
   --    SICHERHEITSHINWEIS: Erlaubt potenzielles Füllen der Datenbank.
   --    Nutzung überwachen oder Rate Limiting / Authentifizierung für Produktionsumgebungen erwägen.
   CREATE POLICY "Allow public insert access" ON public.gantt_shares
     FOR INSERT
     WITH CHECK (true);

   -- 2. Erlaube jedem (anon-Rolle), Zeilen zu lesen (notwendig zum Laden via geteiltem Link/ID)
   --    SICHERHEITSHINWEIS: Basiert auf der Unwahrscheinlichkeit, UUIDs zu erraten. Theoretisch sind alle Zeilen lesbar.
   CREATE POLICY "Allow public read access" ON public.gantt_shares
     FOR SELECT
     USING (true);

   -- Sicherstellen, dass die anon-Rolle die Basis-Rechte hat (normalerweise Standard)
   GRANT SELECT, INSERT ON TABLE public.gantt_shares TO anon;
   -- Explizit unnötige Berechtigungen entziehen (optional, aber empfohlen)
   REVOKE UPDATE, DELETE ON TABLE public.gantt_shares FROM anon;

**4. HTML-Datei konfigurieren:**

* Öffnen Sie die ganttview.html Datei (oder wie auch immer Sie sie genannt haben) in einem Texteditor.
* Suchen Sie den <script>-Block relativ am Ende der Datei.
* Finden Sie die folgenden Zeilen am Anfang des JavaScript-Codes:
    const SUPABASE_URL = 'YOUR_SUPABASE_URL';
    const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
* Gehen Sie zu den Einstellungen Ihres Supabase-Projekts: Project Settings -> API.
* Kopieren Sie die 'Project URL' und fügen Sie sie anstelle von 'YOUR_SUPABASE_URL' ein.
* Kopieren Sie den anon 'public' Schlüssel (im Abschnitt Project API keys) und fügen Sie ihn anstelle von 'YOUR_SUPABASE_ANON_KEY' ein.
* Speichern Sie die HTML-Datei.
* WARNUNG: Committen Sie diese Datei niemals mit Ihren tatsächlichen Zugangsdaten (URL und Key) in ein öffentliches Repository! Diese Konfiguration muss jeder Nutzer für seine eigene Instanz lokal vornehmen.

**5. (Optional) Logo anpassen:**

* Suchen Sie im HTML-Code nach dem <div class="logo-container">-Abschnitt.
* Passen Sie diesen Bereich an, um Ihr eigenes Logo oder einen anderen Titel anzuzeigen.

## Nutzung
Öffnen Sie die (ggf. konfigurierte) ganttview.html-Datei einfach in einem modernen Webbrowser (wie Chrome, Firefox, Edge).

## Disclaimer
Diese Software wird "wie besehen" ("as is") ohne jegliche Gewährleistung, weder ausdrücklich noch stillschweigend, bereitgestellt. Dies umfasst, ist aber nicht beschränkt auf, die Gewährleistung der Marktgängigkeit oder der Eignung für einen bestimmten Zweck.

Die Nutzer sind allein verantwortlich für die Sicherstellung der Einhaltung aller Lizenzbedingungen der in diesem Projekt verwendeten oder referenzierten Abhängigkeiten, insbesondere der dhtmlxGantt-Bibliothek. Die Autoren und Mitwirkenden dieses Projekts übernehmen keine Haftung für Schäden oder Verluste, die aus der Nutzung dieser Software entstehen.
