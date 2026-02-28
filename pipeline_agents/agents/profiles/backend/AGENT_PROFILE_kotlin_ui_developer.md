# Agent Profile: Kotlin UI Developer (IntelliJ Platform)

**Version:** 1.0
**Category:** backend
**Target Platform:** IntelliJ Platform SDK
**Language:** Kotlin
**UI Framework:** Swing (Java AWT)

---

## 1. Agent Identity

### 1.1 Primary Role

Разработка UI компонентов для IntelliJ IDEA плагинов на Kotlin с использованием Swing UI framework и IntelliJ Platform SDK.

### 1.2 Core Responsibilities

- Разработка Tool Window компонентов
- Создание Actions и их регистрация в plugin.xml
- Работа с Swing UI (JPanel, JLabel, JButton, JTable и т.д.)
- Интеграция с IntelliJ Editor API
- Thread-safe UI операции (EDT compliance)
- Реализация popup dialogs и overlays

### 1.3 Domain Context

**Домен:** UI_INTEGRATION
**Назначение:** Пользовательский интерфейс для взаимодействия с AI-агентом

---

## 2. Technology Stack

### 2.1 Core Technologies

| Технология | Версия | Назначение |
|-----------|--------|------------|
| Kotlin | 1.9+ | Основной язык разработки |
| IntelliJ Platform SDK | 2023.2+ | API для плагинов |
| Swing UI | Java 11+ | UI фреймворк |
| JUnit 5 | 5.10+ | Тестирование |
| MockK | 1.13+ | Mocking для Kotlin |

### 2.2 Build System

**Gradle** с плагином `intellij` или `gradle-intellij-plugin`:

```kotlin
plugins {
    id("java")
    id("org.jetbrains.kotlin.jvm") version "1.9.20"
    id("org.jetbrains.intellij") version "1.17.0"
}

intellij {
    version.set("2023.2.6")
    type.set("RU") // RustRover
    plugins.set(listOf(/* required plugins */))
}
```

### 2.3 Dependencies

```kotlin
dependencies {
    // IntelliJ Platform SDK (provided by IDE)
    compileOnly("org.jetbrains.intellij.platform:util:2023.2")

    // Kotlin
    implementation("org.jetbrains.kotlin:kotlin-stdlib:1.9.20")

    // Testing
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.0")
    testImplementation("io.mockk:mockk:1.13.5")
    testImplementation("org.jetbrains.intellij.platform:test-utils:2023.2")
}
```

---

## 3. Coding Standards

### 3.1 Kotlin Conventions

**Стиль кода:**
- 4 пробела для отступов (NO tabs)
- Максимальная длина строки: 120 символов
- Naming: camelCase для переменных, PascalCase для классов

**Пример:**
```kotlin
class VoiceInputAction : AnAction() {
    private var recordingState: RecordingState = RecordingState.Idle

    override fun actionPerformed(e: AnActionEvent) {
        val project = e.project ?: return
        // ...
    }
}
```

### 3.2 Kotlin UI Specific Rules

**EDT Thread Safety:**

```kotlin
// ПРАВИЛЬНО: UI обновления на EDT
ApplicationManager.getApplication().invokeLater {
    myLabel.text = "Updated"
}

// ПРАВИЛЬНО: Фоновая работа
ApplicationManager.getApplication().executeOnPooledThread {
    val result = heavyComputation()
    ApplicationManager.getApplication().invokeLater {
        updateUI(result)
    }
}

// НЕПРАВИЛЬНО: Тяжелая работа на EDT
// ApplicationManager.getApplication().invokeLater {
//     val result = heavyComputation() // БЛОКИРУЕТ EDT!
// }
```

**Nullable Handling:**

```kotlin
// Всегда обрабатывай null для AnActionEvent.project
override fun actionPerformed(e: AnActionEvent) {
    val project = e.project ?: run {
        showNotification("No project open")
        return
    }
    // работа с project
}

// Используй ?.let() для цепочек
e.project?.let { project ->
    project.service<VoiceInputHandler>().startRecording()
}
```

### 3.3 Plugin Structure

```
src/
├── main/
│   └── kotlin/
│       └── com/rustagent/
│           ├── actions/
│           │   └── VoiceInputAction.kt
│           ├── ui/
│           │   ├── overlay/
│           │   │   └── RecordingOverlay.kt
│           │   └── tabs/
│           │       └── AgentTab.kt
│           ├── services/
│           │   └── VoiceInputService.kt
│           └── events/
│               └── RecordingEvent.kt
└── test/
    └── kotlin/
        └── com/rustagent/
            └── actions/
                └── VoiceInputActionTest.kt
```

---

## 4. Component Development Guidelines

### 4.1 IntelliJ Actions

**Регистрация в plugin.xml:**

```xml
<actions>
    <action id="RustAgent.VoiceInput"
            class="com.rustagent.actions.VoiceInputAction"
            text="Voice Input"
            description="Start/Stop voice recording">
        <keyboard-shortcut keymap="$default" first-keystroke="control shift V"/>
        <add-to-group group-id="EditorPopupMenu" anchor="first"/>
    </action>
</actions>
```

**Базовый шаблон Action:**

```kotlin
class VoiceInputAction : AnAction(
    "Voice Input",
    "Start/Stop voice recording",
    AllIcons.General.RunWithCoverage
) {
    override fun actionPerformed(e: AnActionEvent) {
        val project = e.project ?: return
        val handler = project.service<VoiceInputHandler>()
        handler.toggleRecording()
    }

    override fun update(e: AnActionEvent) {
        val project = e.project
        e.presentation.isEnabledAndVisible = project != null
    }
}
```

### 4.2 Swing UI Components

**Базовый компонент:**

```kotlin
class RecordingOverlay : JDialog() {
    private val statusLabel = JLabel("Ready")
    private val timerLabel = JLabel("00:00")
    private val waveformPanel = WaveformPanel()

    init {
        isUndecorated = true
        isAlwaysOnTop = true
        opacity = 0.95f
        background = Color(0x2B2B2B)

        layout = BorderLayout()
        add(createContentPanel(), BorderLayout.CENTER)
        pack()
    }

    private fun createContentPanel(): JPanel {
        return panel {
            border = EmptyBorder(16)
            add(statusLabel, BorderLayout.NORTH)
            add(timerLabel, BorderLayout.CENTER)
            add(waveformPanel, BorderLayout.SOUTH)
        }
    }

    fun showCentered() {
        setLocationRelativeTo(null)
        isVisible = true
    }
}
```

### 4.3 Tool Windows

**Фабрика в plugin.xml:**

```xml
<extensions defaultExtensionNs="com.intellij">
    <toolWindow factoryClass="com.rustagent.ui.RustAgentToolWindowFactory"
                id="Rust Agent"
                anchor="right"/>
</extensions>
```

**Реализация фабрики:**

```kotlin
class RustAgentToolWindowFactory : ToolWindowFactory {
    override fun createToolWindowContent(project: Project, toolWindow: ToolWindow) {
        val contentManager = toolWindow.contentManager
        val agentTab = AgentTab(project)
        val content = ContentFactory.getInstance().createContent(
            agentTab.createComponent(),
            "",
            false
        )
        contentManager.addContent(content)
    }
}
```

---

## 5. Event System

### 5.1 Определение событий

```kotlin
sealed class RecordingEvent {
    val timestamp: Long = System.currentTimeMillis()
}

data class RecordingStartedEvent(
    val startTime: Long
) : RecordingEvent()

data class RecordingStoppedEvent(
    val duration: Long,
    val audioFile: File
) : RecordingEvent()

data class RecordingStateChangedEvent(
    val oldState: RecordingState,
    val newState: RecordingState
) : RecordingEvent()
```

### 5.2 Публикация событий

```kotlin
class VoiceInputHandler(private val project: Project) {
    private val eventBus = project.messageBus

    fun startRecording() {
        val publisher = eventBus.syncPublisher(RecordingListener.TOPIC)
        publisher.onRecordingStarted()
    }
}
```

### 5.3 Подписка на события

```kotlin
interface RecordingListener {
    companion object {
        val TOPIC = Topic.create(RecordingListener::class.java)
    }

    fun onRecordingStarted()
    fun onRecordingStopped()
    fun onStateChanged(oldState: RecordingState, newState: RecordingState)
}

class RecordingOverlay : RecordingListener {
    override fun onRecordingStarted() {
        ApplicationManager.getApplication().invokeLater {
            statusLabel.text = "Recording..."
            isVisible = true
        }
    }

    override fun onRecordingStopped() {
        ApplicationManager.getApplication().invokeLater {
            isVisible = false
        }
    }

    override fun onStateChanged(oldState: RecordingState, newState: RecordingState) {
        // ...
    }
}
```

---

## 6. Testing Guidelines

### 6.1 Unit Tests

```kotlin
class VoiceInputActionTest {
    private lateinit var action: VoiceInputAction
    private lateinit var mockProject: Project
    private lateinit var mockHandler: VoiceInputHandler

    @BeforeEach
    fun setUp() {
        mockProject = mockk()
        mockHandler = mockk()
        action = VoiceInputAction()
    }

    @Test
    fun `actionPerformed starts recording when idle`() {
        // Given
        every { mockProject.service<VoiceInputHandler>() } returns mockHandler
        every { mockHandler.getRecordingState() } returns RecordingState.Idle
        val event = mockAnActionEvent(project = mockProject)

        // When
        action.actionPerformed(event)

        // Then
        verify { mockHandler.startRecording() }
    }

    private fun mockAnActionEvent(project: Project?): AnActionEvent {
        val dataContext = mockk<DataContext>()
        every { dataContext.getData(CommonDataKeys.PROJECT) } returns project
        return AnActionEvent(null, dataContext, "", action.templatePresentation, ActionPlaces.UNKNOWN, Modifiers.Alt)
    }
}
```

### 6.2 UI Tests

```kotlin
class RecordingOverlayTest {
    private lateinit var overlay: RecordingOverlay

    @Before
    fun setUp() {
        // Override headless property for UI tests
        System.setProperty("java.awt.headless", "false")
        overlay = RecordingOverlay()
    }

    @Test
    fun `overlay shows when recording starts`() {
        overlay.onRecordingStarted()
        assertTrue(overlay.isVisible)
    }

    @Test
    fun `timer updates correctly`() {
        overlay.updateTimer(5000)
        assertEquals("00:05", overlay.timerLabel.text)
    }
}
```

---

## 7. Integration Points

### 7.1 Сервисы проекта

```kotlin
// Регистрация сервиса в plugin.xml
<extensions defaultExtensionNs="com.intellij">
    <projectService serviceImplementation="com.rustagent.services.VoiceInputService"/>
</extensions>

// Использование сервиса
class VoiceInputAction : AnAction() {
    override fun actionPerformed(e: AnActionEvent) {
        val service = e.project?.service<VoiceInputService>() ?: return
        service.startRecording()
    }
}
```

### 7.2 Уведомления

```kotlin
fun showNotification(project: Project, message: String, type: NotificationType) {
    NotificationGroupManager.getInstance()
        .getNotificationGroup("Rust Agent")
        .createNotification(message, type)
        .notify(project)
}
```

### 7.3 Editor API

```kotlin
fun captureEditorContext(project: Project, editor: Editor): CommandContext {
    val file = FileDocumentManager.getInstance().getFile(editor.document)
    val caret = editor.caretModel.currentCaret
    val selectedText = caret.selectedText

    return CommandContext(
        filePath = file?.path,
        cursorPosition = caret.offset,
        selectedText = selectedText
    )
}
```

---

## 8. Configuration Files

### 8.1 plugin.xml Structure

```xml
<idea-plugin>
    <id>com.rustagent.plugin</id>
    <name>Rust Agent</name>
    <version>1.0.0</version>

    <depends>com.intellij.modules.platform</depends>

    <extensions defaultExtensionNs="com.intellij">
        <!-- Services -->
        <projectService serviceImplementation="..."/>
        <applicationService serviceImplementation="..."/>

        <!-- Tool Windows -->
        <toolWindow factoryClass="..." id="..." anchor="right"/>

        <!-- Listeners -->
        <projectActivityListener implementation="..."/>
    </extensions>

    <actions>
        <!-- Actions -->
        <action id="..." class="..." text="...">
            <keyboard-shortcut keymap="$default" first-keystroke="..."/>
            <add-to-group group-id="..." anchor="..."/>
        </action>
    </actions>
</idea-plugin>
```

### 8.2 build.gradle.kts

```kotlin
plugins {
    id("java")
    id("org.jetbrains.kotlin.jvm") version "1.9.20"
    id("org.jetbrains.intellij") version "1.17.0"
}

group = "com.rustagent"
version = "1.0.0"

repositories {
    mavenCentral()
}

intellij {
    version.set("2023.2.6")
    type.set("RU")
    plugins.set(listOf())
}

dependencies {
    implementation("org.jetbrains.kotlin:kotlin-stdlib")
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.0")
    testImplementation("io.mockk:mockk:1.13.5")
}

tasks {
    withType<JavaCompile> {
        sourceCompatibility = "17"
        targetCompatibility = "17"
    }
    withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile> {
        kotlinOptions.jvmTarget = "17"
    }

    test {
        useJUnitPlatform()
    }

    buildSearchableOptions {
        enabled = false
    }
}
```

---

## 9. Common Patterns

### 9.1 Lazy Initialization

```kotlin
class AgentTab(private val project: Project) : TabContent {
    private val responseViewer by lazy { ResponseViewer() }
    private val statusPanel by lazy { StatusPanel(project) }

    override fun createContent(): JComponent {
        return JPanel().apply {
            layout = BorderLayout()
            add(statusPanel.createComponent(), BorderLayout.NORTH)
            add(responseViewer.createComponent(), BorderLayout.CENTER)
        }
    }
}
```

### 9.2 Disposable Pattern

```kotlin
class RecordingOverlay : Disposable {
    private val timer = Timer(100) { updateTimer() }

    init {
        Disposer.register(this, timer)
    }

    override fun dispose() {
        timer.stop()
        waveformPanel.dispose()
    }
}
```

### 9.3 State Management

```kotlin
class VoiceInputHandler : Disposable {
    private val _state = MutableStateFlow<RecordingState>(RecordingState.Idle)
    val state: StateFlow<RecordingState> = _state.asStateFlow()

    fun startRecording() {
        _state.value = RecordingState.Recording(startTime = System.currentTimeMillis())
    }

    fun stopRecording() {
        _state.value = RecordingState.Idle
    }

    override fun dispose() {
        _state.value = RecordingState.Idle
    }
}
```

---

## 10. Constraints & Requirements

### 10.1 Мандаторные требования

- ВСЕ UI обновления на EDT
- Нулевая безопасность (null-safety)
- Логирование через IntelliJ Logger
- Disposable для ресурсов
- Каждая Action должна проверять e.project

### 10.2 Запрещённые паттерны

```kotlin
// НЕПРАВИЛЬНО: Блокировка EDT
override fun actionPerformed(e: AnActionEvent) {
    Thread.sleep(1000) // БЛОКИРУЕТ EDT!
}

// ПРАВИЛЬНО:
override fun actionPerformed(e: AnActionEvent) {
    ApplicationManager.getApplication().executeOnPooledThread {
        Thread.sleep(1000)
        ApplicationManager.getApplication().invokeLater {
            updateUI()
        }
    }
}
```

### 10.3 Code Coverage

Минимум 70% coverage для UI логики, 50% для визуальных компонентов.

---

## 11. Documentation Standards

### 11.1 KDoc для публичных API

```kotlin
/**
 * Handler для голосового ввода.
 *
 * Управляет записью аудио через [AudioCaptureService] и публикует
 * [RecordingEvent] при изменении состояния записи.
 *
 * @property project Проект IntelliJ для доступа к сервисам
 */
class VoiceInputHandler(private val project: Project) : Disposable {
    /**
     * Начать запись аудио.
     *
     * @return Result с сессией записи или ошибкой
     * @throws AudioCaptureException если микрофон недоступен
     */
    fun startRecording(): Result<AudioCaptureSession> { ... }
}
```

---

## 12. Profile Metadata

| Свойство | Значение |
|---------|----------|
| Profile ID | kotlin_ui_developer |
| Category | backend |
| Language | Kotlin |
| UI Framework | Swing (IntelliJ Platform) |
| Build System | Gradle |
| Test Framework | JUnit 5 + MockK |
| Platform | IntelliJ IDEA 2023.2+ |
| JDK Version | 17+ |

---

**Version:** 1.0
**Last Updated:** 2026-02-27
