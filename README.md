# NXLD 链式加载流程文档 / Chain Loading Workflow Documentation / Kettenladungs-Workflow-Dokumentation

本文档描述了重构后的 NXLD 指针传递插件的链式加载机制和完整工作流程。
This document describes the chain loading mechanism and complete workflow of the refactored NXLD pointer transfer plugin.
Dieses Dokument beschreibt den Kettenladungsmechanismus und den vollständigen Workflow des refactorierten NXLD-Zeigerübertragungs-Plugins.

## 架构设计 / Architecture Design / Architektur-Design

### 核心原则 / Core Principles / Kernprinzipien

1. **去中心化配置 / Decentralized Configuration / Dezentralisierte Konfiguration**
   - 每个插件拥有自己的 `.nxpt` 配置文件 / Each plugin has its own `.nxpt` configuration file / Jedes Plugin hat seine eigene `.nxpt`-Konfigurationsdatei
   - 配置规则分散在各个插件的配置文件中 / Configuration rules are distributed across plugin configuration files / Konfigurationsregeln sind über Plugin-Konfigurationsdateien verteilt

2. **链式加载机制 / Chain Loading Mechanism / Kettenladungsmechanismus**
   - PointerTransferPlugin 启动时只加载入口插件的 `.nxpt` 文件 / PointerTransferPlugin only loads entry plugin's `.nxpt` file on startup / PointerTransferPlugin lädt beim Start nur die `.nxpt`-Datei des Einstiegs-Plugins
   - 根据入口插件的规则链式加载其他插件的 `.nxpt` 文件 / Chain load other plugins' `.nxpt` files according to entry plugin's rules / Kettenweise andere Plugins' `.nxpt`-Dateien gemäß Einstiegs-Plugin-Regeln laden

3. **延迟加载插件 / Lazy Plugin Loading / Lazy Plugin-Ladung**
   - 插件只在需要时加载，而非在加载 `.nxpt` 时加载 / Plugins are loaded only when needed, not when loading `.nxpt` / Plugins werden nur bei Bedarf geladen, nicht beim Laden von `.nxpt`
   - 根据条件传递暂缓启动，只在实际传递时进行启动插件 / Defer plugin startup based on transfer conditions, only start plugins during actual transfer / Plugin-Start basierend auf Übertragungsbedingungen verzögern, Plugins nur bei tatsächlicher Übertragung starten

## 完整工作流程 / Complete Workflow / Vollständiger Workflow

```mermaid
graph TB

    subgraph Phase1[阶段1: 初始化 / Phase 1: Initialization / Phase 1: Initialisierung]

        A1[引擎启动<br/>Engine Start<br/>Engine-Start] --> A2[加载PointerTransferPlugin<br/>Load PointerTransferPlugin<br/>PointerTransferPlugin laden]

        A2 --> A3[读取pointer_transfer_plugin.nxpt<br/>Read pointer_transfer_plugin.nxpt<br/>pointer_transfer_plugin.nxpt lesen]

        A3 --> A4[解析入口插件配置<br/>Parse Entry Plugin Config<br/>Einstiegs-Plugin-Konfiguration parsen]

        A4 --> A5[加载entry_plugin.nxpt<br/>Load entry_plugin.nxpt<br/>entry_plugin.nxpt laden]

        A5 --> A6[链式加载相关插件.nxpt<br/>Chain Load Related Plugin .nxpt<br/>Verwandte Plugin-.nxpt kettenweise laden]

    end

    

    subgraph Phase2[阶段2: 数据传递 / Phase 2: Data Transfer / Phase 2: Datenübertragung]

        B1[EntryPlugin自举<br/>EntryPlugin Self-bootstrap<br/>EntryPlugin Selbststart] --> B2[调用EntryPoint 42<br/>Call EntryPoint 42<br/>EntryPoint 42 aufrufen]

        B2 --> B3[PointerTransfer规则0<br/>→ AddPlugin.Add参数0<br/>PointerTransfer Rule 0<br/>→ AddPlugin.Add Parameter 0<br/>PointerTransfer Regel 0<br/>→ AddPlugin.Add Parameter 0]

        B4[EntryPlugin自举<br/>EntryPlugin Self-bootstrap<br/>EntryPlugin Selbststart] --> B5[调用EntryPoint 58<br/>Call EntryPoint 58<br/>EntryPoint 58 aufrufen]

        B5 --> B6[PointerTransfer规则1<br/>→ AddPlugin.Add参数1<br/>PointerTransfer Rule 1<br/>→ AddPlugin.Add Parameter 1<br/>PointerTransfer Regel 1<br/>→ AddPlugin.Add Parameter 1]

    end

    

    subgraph Phase3[阶段3: 计算执行 / Phase 3: Calculation Execution / Phase 3: Berechnungsausführung]

        C1[延迟加载AddPlugin<br/>Lazy Load AddPlugin<br/>AddPlugin verzögert laden] --> C2[AddPlugin接收参数<br/>AddPlugin Receive Parameters<br/>AddPlugin Parameter empfangen]

        C2 --> C3[计算: 42+58=100<br/>Calculate: 42+58=100<br/>Berechnen: 42+58=100]

        C3 --> C4[格式化结果字符串<br/>Format Result String<br/>Ergebniszeichenfolge formatieren]

        C4 --> C5[触发主动调用规则<br/>Trigger Active Call Rule<br/>Aktive Aufrufregel auslösen]

    end

    

    subgraph Phase4[阶段4: 结果输出 / Phase 4: Result Output / Phase 4: Ergebnisausgabe]

        D1[PointerTransfer规则1<br/>主动调用规则<br/>PointerTransfer Rule 1<br/>Active Call Rule<br/>PointerTransfer Regel 1<br/>Aktive Aufrufregel] --> D2[自动获取GetResultString<br/>Auto-get GetResultString<br/>GetResultString automatisch abrufen]

        D2 --> D3[加载add_plugin.nxpt<br/>Load add_plugin.nxpt<br/>add_plugin.nxpt laden]

        D3 --> D4[规则0: 设置日志级别=INFO<br/>Rule 0: Set Log Level=INFO<br/>Regel 0: Protokollierungsebene=INFO setzen]

        D4 --> D5[延迟加载FileLoggerPlugin<br/>Lazy Load FileLoggerPlugin<br/>FileLoggerPlugin verzögert laden]

        D5 --> D6[加载file_logger_plugin.nxpt<br/>Load file_logger_plugin.nxpt<br/>file_logger_plugin.nxpt laden]

        D6 --> D7[写入日志文件<br/>Write to Log File<br/>In Protokolldatei schreiben]

    end

    

    Phase1 --> Phase2

    Phase2 --> Phase3

    Phase3 --> Phase4

    

    style Phase1 fill:#E3F2FD

    style Phase2 fill:#FFF3E0

    style Phase3 fill:#E8F5E9

    style Phase4 fill:#F3E5F5

```

## 配置文件结构 / Configuration File Structure / Konfigurationsdatei-Struktur

### pointer_transfer_plugin.nxpt
```ini
[EntryPlugin]
PluginName=EntryPlugin
PluginPath=./plugins/entry_plugin.dll
NxptPath=./plugins/entry_plugin.nxpt
```

### entry_plugin.nxpt
包含从 EntryPlugin 到其他插件的传递规则：
- TransferRule_0: EntryPlugin.EntryPoint[0] → AddPlugin.Add[0]
- TransferRule_1: EntryPlugin.EntryPoint[0] → AddPlugin.Add[1]
- TransferRule_2: EntryPlugin.EntryPoint[0] → FileLoggerPlugin.Write[1]
- TransferRule_3: FileLoggerPlugin.Write[0] = 2 (常量值)

### add_plugin.nxpt
包含从 AddPlugin 到其他插件的传递规则：
- TransferRule_0: AddPlugin.GetResultString[0] → FileLoggerPlugin.Write[1] (条件: not_null)
- TransferRule_1: AddPlugin.Add (主动调用规则) → FileLoggerPlugin.Write[1]

### file_logger_plugin.nxpt
包含 FileLoggerPlugin 的配置规则：
- TransferRule_0: FileLoggerPlugin.Write[0] = 2 (常量值)

## 链式加载流程 / Chain Loading Flow / Kettenladungsfluss

1. **启动阶段 / Startup Phase / Startphase**
   - PointerTransferPlugin 加载自己的 `.nxpt` 文件
   - 解析入口插件配置，获取入口插件路径和 `.nxpt` 路径
   - 加载入口插件的 `.nxpt` 文件

2. **链式加载阶段 / Chain Loading Phase / Kettenladungsphase**
   - 遍历入口插件 `.nxpt` 中的所有规则
   - 对于每个规则的目标插件，检查其 `.nxpt` 文件是否已加载
   - 如果未加载，则根据插件路径构建 `.nxpt` 路径并加载
   - 递归处理新加载的规则中的目标插件

3. **延迟加载阶段 / Lazy Loading Phase / Lazy-Ladungsphase**
   - 当需要调用目标插件接口时，检查该插件的 `.nxpt` 文件是否已加载
   - 如果未加载，则先加载其 `.nxpt` 文件
   - 然后加载插件本身并调用接口

## 优势 / Advantages / Vorteile

1. **模块化 / Modularity / Modularität**
   - 每个插件管理自己的配置规则，便于维护 / Each plugin manages its own configuration rules, easy to maintain / Jedes Plugin verwaltet seine eigenen Konfigurationsregeln, einfach zu warten

2. **可扩展性 / Scalability / Skalierbarkeit**
   - 添加新插件只需创建对应的 `.nxpt` 文件 / Adding new plugins only requires creating corresponding `.nxpt` file / Hinzufügen neuer Plugins erfordert nur das Erstellen der entsprechenden `.nxpt`-Datei

3. **性能优化 / Performance Optimization / Leistungsoptimierung**
   - 延迟加载机制减少启动时间 / Lazy loading mechanism reduces startup time / Lazy-Ladungsmechanismus reduziert Startzeit
   - 只加载实际使用的插件 / Only load actually used plugins / Nur tatsächlich verwendete Plugins laden

4. **解耦设计 / Decoupling Design / Entkopplungsdesign**
   - 插件之间通过配置文件解耦 / Plugins are decoupled through configuration files / Plugins sind über Konfigurationsdateien entkoppelt
   - 修改一个插件的配置不影响其他插件 / Modifying one plugin's configuration doesn't affect other plugins / Änderung der Konfiguration eines Plugins beeinflusst andere Plugins nicht

