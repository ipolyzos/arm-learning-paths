---
title: Windows Performance Analyzer (WPA) plugin

minutes_to_complete: 15

official_docs: https://github.com/arm-developer-tools/windowsperf-wpa-plugin

author_primary: Alaaeddine Chakroun 

### Optional additional search terms (one per line) to assist in finding the article
additional_search_terms:
  - perf
  - profiling
  - profiler
  - windows
  - woa
  - windows on arm
  - windows performance analyzer
  - wpa
  
### FIXED, DO NOT MODIFY
weight: 1 # Defines page ordering. Must be 1 for first (or only) page.
tool_install: true # Set to true to be listed in main selection page, else false
multi_install: FALSE # Set to true if first page of multi-page article, else false
multitool_install_part: false # Set to true if a sub-page of a multi-page article, else false
layout: installtoolsall # DO NOT MODIFY. Always true for tool install articles
---

## What is the Windows Performance Analyzer plugin?

The Windows Performance Analyzer (WPA) plugin connects [WindowsPerf](/learning-paths/laptops-and-desktops/windowsperf/) to the Windows Performance Analyzer. Windows Perf is a lightweight performance profiling tool inspired by Linux Perf and designed for Windows on Arm.

Windows Performance Analyzer provides developers with diagnostics and performance tuning. It generates data tables and graphs of Event Tracing for Windows (ETW) events, which are recorded in one of three ways:
- Windows Performance Recorder (WPR)
- Xperf
- or through an assessment that's run in the Assessment Platform.
 
WPA can open event trace log (ETL) files, which you can use for analysis.

The WPA plugin is built using the [Microsoft Performance Toolkit SDK](https://github.com/microsoft/microsoft-performance-toolkit-sdk), a collection of tools to create and extend performance analysis applications. The plugin parses JSON output from Windows Perf so that it can be visualized in WPA. 

## What are some of the features of the WPA plugin? 

The WindowsPerf GUI extension includes features, which are designed to streamline the user experience:

### Timeline view

The timeline view visualizes the `wperf stat` timeline data plotted by event group:

![Timeline By Core Table](/install-guides/_images/wpa-timeline-by-core.png)

### Telemetry view

The telemetry view displays telemetry events grouped by unit:

![Telemetry Table](/install-guides/_images/wpa-telemetry-table.png)

## How do I install the WPA plugin?

Before installing the plugin, you need to install WPA.

### Install WPA

For Windows on Arm devices, you can install WPA from the Microsoft Store.

Open the Microsoft Store and search for "windows performance analyzer".

![WPA store](/install-guides/_images/wpa-store.png)

Hover over the card, and the "Free" button will become a "Get" button. Click the "Get" button to start installation. 

Wait for WPA to be installed, and launch it from the Windows menu.

![WPA installation #center](/install-guides/_images/wpa-installation.png)

{{% notice Note %}}
The WPA plugin requires WPA version `11.0.7.2` or higher. You can check the version by clicking `Help` -> `About Windows Performance Analyzer`.
{{% /notice %}}

Close Windows Performance Analyzer.

### Install the WPA plugin

You are ready to install the WPA plugin, which is a single `.dll` file.

Download the `.zip` file from the [Windows Perf WPA plugin GitHub releases page](https://github.com/arm-developer-tools/windowsperf-wpa-plugin/releases) on GitHub.

Alternatively, you can download the `.zip` file from a Command Prompt:

```console
mkdir wpa-plugin
cd wpa-plugin
curl -L -O https://github.com/arm-developer-tools/windowsperf-wpa-plugin/releases/download/1.0.2/wpa-plugin-1.0.2.zip
```

Extract the `.dll` file from the downloaded `.zip` file. 

```console
tar -xmf wpa-plugin-1.0.2.zip
```

The file `WPAPlugin.dll` is now in your `wpa-plugin` directory. 

There are three ways you can use the `WPAPlugin.dll` file: 

#### 1. Start WPA from the command line and pass the plugin directory location using a flag.

Use the `-addsearchdir` flag to tell `wpa` where to find plugins.

For example, if you downloaded the DLL in your `Downloads` directory, you can run `wpa` as shown below:

```bash
wpa -addsearchdir %USERPROFILE%\Downloads\wpa-plugin-1.0.2
```
        
#### 2. Set an environment variable.

Set the `WPA_ADDITIONAL_SEARCH_DIRECTORIES` environment variable to the location of the `.dll` file.

#### 3. Copy the .dll file to the CustomDataSources directory next to the WPA executable.

The default location is: 
        `C:\\Program Files (x86)\Windows Kits\10\Windows Performance Toolkit\CustomDataSources`

## How can I verify the WPA plugin is installed?

To verify the plugin is loaded, launch WPA and the plugin should appear on the Installed Plugins list.

![WPA installation confirmation](/install-guides/_images/wpa-install-plugin.png)

## How can I run the WPA plugin from the command line?

To open a JSON file directly from the command line, you can use the `-i` flag to specify the file path to open.

For example: to open `timeline_long.json` in your downloads directory, run the command:

```console
wpa -addsearchdir %USERPROFILE%\Downloads\wpa-plugin-1.0.2 -i %USERPROFILE%\Downloads\timeline_long.json
```
## How do I uninstall the WPA plugin?

To uninstall the plugin simply delete the `WPAPlugin.dll` file.
