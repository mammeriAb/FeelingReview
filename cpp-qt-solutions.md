# C++ / Qt Exercise Solutions

Answers for [cpp-qt-exercises.md](cpp-qt-exercises.md). Try the exercise first.

---

## 1. C++ core

### Exercise 1

| Line | Result |
|------|--------|
| (a) | **Error.** A reference must be initialized. |
| (b) | **OK.** Pointers can be null then reseated. |
| (c) | **Error** (if `r` was not initialized). You cannot declare `int& r;` at all. |
| (d) | **OK.** `r2` aliases `x`. |
| (e) | **OK.** Changes `x` to 7 (does not reseat `r2`). |
| (f) | **OK.** Pointers may be null. |
| (g) | **Error.** `r2` is an `int&`; `nullptr` is not an `int`. |

### Exercise 2

```
const int* a        *a = 1?  no     a = &n?  yes   // pointer to const
int* const b        *b = 1?  yes    b = &n?  no    // const pointer
const int* const c  *c = 1?  no     c = &n?  no
```

(`n` itself is non-const, so a non-const pointer to `n` may write. `a`/`c` forbid writes through that pointer.)

### Exercise 3

`s` is destroyed when `name()` returns, so `p` dangles. Undefined behavior.

```cpp
std::string name() { return "Qt"; }

int main() {
    std::string s = name();
    std::cout << s;
}
```

### Exercise 4

```cpp
class File {
    FILE* f_ = nullptr;
public:
    explicit File(const char* path, const char* mode) {
        f_ = std::fopen(path, mode);
        if (!f_) throw std::runtime_error("open failed");
    }
    ~File() { if (f_) std::fclose(f_); }

    File(const File&) = delete;
    File& operator=(const File&) = delete;

    File(File&& o) noexcept : f_(o.f_) { o.f_ = nullptr; }
    File& operator=(File&& o) noexcept {
        if (this != &o) {
            if (f_) std::fclose(f_);
            f_ = o.f_;
            o.f_ = nullptr;
        }
        return *this;
    }

    FILE* get() const { return f_; }
};
```

### Exercise 5

```cpp
class Buffer {
    std::vector<char> data_;
public:
    explicit Buffer(std::size_t n) : data_(n) {}
};
```

`vector` already copies, moves, and frees. That is Rule of Zero.

### Exercise 6

`b` is `"hello"`. `a` is a **valid unspecified state** (often empty, but the standard does not require empty). Do not assume `a.empty()`.

`std::move` is only a cast; the move constructor of `string` does the steal.

### Exercise 7

Typical print: `B` `D` `~B`. **`~D` may not run.** Deleting `Derived` through a non-virtual `Base` destructor is **undefined behavior**.

Fix:

```cpp
virtual ~Base() { std::puts("~B"); }
```

Then: `B` `D` `~D` `~B`.

### Exercise 8

- `d.f(1)`: `1` converts to `double` → prints `D` (`B::f(int)` is **hidden**).
- `b.f(1)`: virtual call to `B::f(int)` (not overridden) → prints `B`.

Fix:

```cpp
struct D : B {
    void f(int) override { std::puts("D"); }
};
```

### Exercise 9

Prints `animal` (slicing: `print` takes `Animal` **by value**).

```cpp
void print(const Animal& a) { std::puts(a.name()); }
```

### Exercise 10

(a) **error** — implicit `int` → `Meter` blocked.  
(b) **OK** — explicit construction.  
(c) **error** — copy-init is implicit.

`explicit` stops `take(10)` from silently inventing a `Meter`.

---

## 2. Modern C++ and STL

### Exercise 11

1. `unique_ptr` (or a parented `QObject` on the stack/member).  
2. `shared_ptr`.  
3. `weak_ptr` (breaks the cycle).  
4. Stack object: `std::mutex m;` plus `lock_guard` — never `unique_ptr<mutex>` unless the mutex itself is heap-allocated.

### Exercise 12

Each node holds a `shared_ptr` to the other → refcount never 0.

```cpp
std::weak_ptr<Node> parent;  // child does not keep parent alive
std::shared_ptr<Node> child;
```

### Exercise 13

`[&]` captures `x` by reference; `x` is dead when the lambda runs.

```cpp
return [=] { return x + 1; };
return [x] { return x + 1; };
return [y = x] { return y + 1; };   // init-capture
```

### Exercise 14

`push_back` **reallocates** if `size == capacity` → all iterators dangle. If there was spare capacity, `begin()` stays valid.

`v.reserve(v.size() + 1);` (or a large enough reserve up front) avoids reallocation.

### Exercise 15

1. `void f(std::string_view s);` or `const std::string&`  
2. `void setName(std::string s) { name_ = std::move(s); }`  
3. `void f(std::uint32_t id);` (by value)  
4. `void f(std::unique_ptr<Widget> w);` if it always takes ownership, **or** `unique_ptr<Widget>&` / raw `Widget*` if it might not.

### Exercise 16

```cpp
return std::make_unique<Widget>(std::forward<Args>(args)...);
```

(`make_unique` already forwards; the explicit `forward` is what the exercise wants if you used `new Widget(...)`.)

Equivalent: `std::unique_ptr<Widget>(new Widget(std::forward<Args>(args)...));` — prefer `make_unique`.

### Exercise 17

```cpp
std::lock_guard<std::mutex> lock(g_mutex);
if (error)
    return;
use_shared_data();
```

### Exercise 18

```cpp
std::optional<User> find_user(const std::vector<User>& users, int id) {
    for (const auto& u : users)
        if (u.id == id) return u;
    return std::nullopt;
}

if (auto u = find_user(users, 42))
    use(*u);
```

To avoid a copy: return `const User*` or `optional<std::reference_wrapper<const User>>`.

---

## 3. Qt — QObject, signals, slots

### Exercise 19

Missing `Q_OBJECT` in the class body (and a moc’d header).

Symptoms: linker **undefined vtable / metaObject**, signals are not real methods, `connect` / properties / `qobject_cast` fail.

```cpp
class Counter : public QObject {
    Q_OBJECT
    // ...
};
```

### Exercise 20

- **`b`:** parented to `Window` → deleted in `~QObject`.  
- **`c`:** **leak** unless you parent it, `delete` it, or `unique_ptr`.  
- **`d`:** parented, then **manually deleted**. That is OK: `delete` unregisters from the parent. Do not delete it **again**.  
- Prefer not mixing styles: either all parented or all smart pointers, never **both** on the same object.

### Exercise 21

The slot is on the call stack of the button/window. Immediate `delete this` can use-after-free when the slot returns.

```cpp
this->deleteLater();
```

(Or `Qt::WA_DeleteOnClose` and `close()`.)

### Exercise 22

```cpp
connect(btn, &QPushButton::clicked, this, &Window::onGo);

connect(btn, &QPushButton::clicked, this, [] {
    qDebug() << "go";
});
```

The third argument `this` is the **context object**: the lambda is dropped when `Window` is destroyed.

### Exercise 23

1. **Thread B** (queued, because affinity differs).  
2. Slot runs **immediately in thread A** — unsafe if it touches B-only objects.  
3. `Q_DECLARE_METATYPE(MyStruct)` and `qRegisterMetaType<MyStruct>()` so the copy can be posted.

### Exercise 24

- `a` **valid** — exact match.  
- `b` **valid** — trailing extra signal args are discarded.  
- `c` **invalid** — slot wants more arguments than the signal.  
- `d` **invalid** — types/order do not match.

### Exercise 25

```cpp
connect(searchBox, &SearchBox::textChanged,
        app, &App::queryChanged);
```

### Exercise 26

1. Make `Sample` copyable (it is).  
2. `Q_DECLARE_METATYPE(Sample)` (typically next to the type).  
3. At startup: `qRegisterMetaType<Sample>();`  
4. Signal: `void sampled(Sample);` emit **by value** (or `QSharedPointer<const Sample>` if large).

### Exercise 27

`b` is a valid `QPushButton*`. `l` is **`nullptr`**.

`qobject_cast` uses the meta-object and returns null on mismatch. `static_cast<QLabel*>(o)` would be a **wrong** pointer (UB if used).

### Exercise 28

```cpp
class Counter : public QObject {
    Q_OBJECT
    Q_PROPERTY(int count READ count WRITE setCount NOTIFY countChanged)
    int m_count = 0;
public:
    int count() const { return m_count; }
    void setCount(int v) {
        if (m_count == v) return;
        m_count = v;
        emit countChanged(m_count);
    }
signals:
    void countChanged(int);
};
```

### Exercise 29

```cpp
QString text = QString::fromUtf8(bytes);
QByteArray out = text.toUtf8();
```

Latin-1 maps bytes 0–255 to U+0000–U+00FF. UTF-8 multi-byte sequences would become garbage.

---

## 4. Event loop and widgets

### Exercise 30

1. `QCoreApplication`  
2. `QGuiApplication`  
3. `QApplication`

### Exercise 31

```cpp
lineEdit->installEventFilter(this);

bool Window::eventFilter(QObject* obj, QEvent* ev) {
    if (obj == lineEdit && ev->type() == QEvent::KeyPress) {
        auto* k = static_cast<QKeyEvent*>(ev);
        if (k->key() == Qt::Key_Escape) {
            lineEdit->clear();
            return true; // consume
        }
    }
    return QWidget::eventFilter(obj, ev);
}
```

### Exercise 32

The layout is never installed on `w`, and `setGeometry` is not a substitute for a layout.

```cpp
auto* w = new QWidget;
auto* layout = new QVBoxLayout(w);   // parent = widget
layout->addWidget(new QPushButton("A"));
```

or `w->setLayout(layout);`

### Exercise 33

```cpp
void MainWindow::closeEvent(QCloseEvent* e) {
    if (!m_dirty) { e->accept(); return; }
    auto r = QMessageBox::question(this, tr("Quit"),
                                   tr("Discard unsaved changes?"));
    if (r == QMessageBox::Yes) e->accept();
    else e->ignore();
}
```

(`QMessageBox` here uses a nested loop — acceptable for a simple close prompt.)

### Exercise 34

```cpp
// member: QTimer* m_search { new QTimer(this) };
m_search->setSingleShot(true);
m_search->setInterval(300);
connect(m_search, &QTimer::timeout, this, [this, edit] {
    runSearch(edit->text());
});
connect(edit, &QLineEdit::textChanged, this, [this] {
    m_search->start();   // restarts the 300 ms
});
```

### Exercise 35

Call `update()` (or `update(rect)`). It **schedules** a coalesced paint.

Create `QPainter` only inside `paintEvent(QPaintEvent*)`. `repaint()` forces an immediate paint — usually unnecessary and can be slower.

### Exercise 36

Problems: nested event loop (`exec`) lets the user re-enter `onExport`; dialog is leaked (`new` without `WA_DeleteOnClose` / parent delete).

Safer: `dlg->setWindowModality(Qt::WindowModal); dlg->open();` plus `Qt::WA_DeleteOnClose`, disable the Export action while running, or run work on a thread and close the dialog on a `finished` signal.

---

## 5. Model/View

### Exercise 37

```cpp
class NameModel : public QAbstractListModel {
    Q_OBJECT
    QStringList m_names;
public:
    int rowCount(const QModelIndex& parent = {}) const override {
        if (parent.isValid()) return 0;
        return m_names.size();
    }
    QVariant data(const QModelIndex& index, int role) const override {
        if (!index.isValid() || role != Qt::DisplayRole) return {};
        return m_names.at(index.row());
    }
};
```

### Exercise 38

```cpp
void NameModel::append(const QString& name) {
    const int row = m_names.size();
    beginInsertRows(QModelIndex{}, row, row);
    m_names.append(name);
    endInsertRows();
}
```

Skipping `beginInsertRows` means views never know the structure changed → missing rows, crashes, desynced delegates.

### Exercise 39

`Qt::DecorationRole` (icon) and `Qt::ToolTipRole`. Optionally `Qt::EditRole` if the cell is editable.

---

## 6. Threading

### Exercise 40

`w->work()` is a **normal C++ call** on the GUI stack. `moveToThread` only changes where **queued events/slots** run.

```cpp
connect(t, &QThread::started, w, &Worker::work);
t->start();
// or:
QMetaObject::invokeMethod(w, &Worker::work, Qt::QueuedConnection);
```

### Exercise 41

The constructor (and `new QTimer(this)`) still runs in the **GUI** thread. The timer’s affinity is the GUI thread; `moveToThread` after that is too late for “create children in constructor”.

Create the timer in `work()` / a `start()` slot **after** the object lives on `thread`, or call `moveToThread` **before** creating children (constructor still runs on the creating thread — so don’t create QObject children in that constructor).

### Exercise 42

```cpp
class Worker : public QObject {
    Q_OBJECT
    std::atomic<bool> stop_{false};
public slots:
    void run() {
        while (!stop_) { /* chunk of work */ }
    }
    void stop() { stop_ = true; }
};

// GUI:
QMetaObject::invokeMethod(&worker, &Worker::stop, Qt::QueuedConnection);
thread.quit();
thread.wait();
```

If the worker has its own event loop, `stop()` can `QThread::currentThread()->quit();` instead of a flag.

### Exercise 43

Widgets must be used only on the **GUI thread**.

```cpp
// Worker:
signals:
    void finished();
// GUI:
connect(worker, &Worker::finished, this, [] {
    QMessageBox::information(this, tr("Done"), tr("Finished"));
});
```

### Exercise 44

`processEvents` can deliver another click into `onExport`, destroy widgets, or re-enter the same function while locals are live.

Better: run `heavyStep` in a worker / `QtConcurrent::run`, emit `progress(i)`, GUI slot updates a `QProgressBar`. No nested pumping.

---

## 7. QML

### Exercise 45

Context property:

```cpp
engine.rootContext()->setContextProperty("counter", &counter);
```

```qml
Button { onClicked: counter.increment() }
```

Instantiable type:

```cpp
qmlRegisterType<Counter>("App", 1, 0, "Counter");
// or QML_ELEMENT + qt_add_qml_module
```

```qml
import App 1.0
Counter { id: c }
```

### Exercise 46

The `Text` stays at the old value forever (binding does not re-run).

Emit `countChanged` from the setter when the value actually changes (see Exercise 28).

### Exercise 47

By default QML/JS **takes ownership** of a `QObject*` returned to QML → QML will `delete` it. If C++ also deletes it → **double free**.

```cpp
QQmlEngine::setObjectOwnership(item, QQmlEngine::CppOwnership);
```

Context properties are CppOwnership by default; factory returns are usually JavaScriptOwnership.

### Exercise 48

Each value writes the other → **binding loop** (Qt warns, values fight).

```qml
// C++ or QML model:
// property int value
SpinBox { value: model.value; onValueModified: model.value = value }
SpinBox { value: model.value; onValueModified: model.value = value }
```

One source of truth; both controls write **to the model**, not to each other.

---

## 8. Mixed scenarios

### Exercise 49

- **GUI:** `MainWindow`, `QTableView`, `QStandardItemModel` or custom model (GUI thread).  
- **Worker:** `QObject` in a `QThread` (or `QtConcurrent::run`) that reads/parses the file.  
- **Signals:** `loadRequested(path)` queued to worker; `rowsReady(...)` / `failed(QString)` queued back. Model updated only in GUI slots.  
- **Ownership:** worker parented to `nullptr`, `moveToThread`; `connect(thread, &QThread::finished, worker, &QObject::deleteLater)`.  
- **Close mid-load:** `QPointer<MainWindow>` in the lambda **or** context object `this` on the connection; worker `stop` flag; ignore results if the pointer is null. Do not touch the view from the worker.

### Exercise 50

1. **`unique_ptr` + parent `this`** on the same `QPushButton` → double-delete.  
2. **Old `SIGNAL(clicked())` connected to `SLOT(onGo(int))`** — argument mismatch; fails at runtime (or does nothing). PMF would not compile.  
3. **`worker_.moveToThread`** while `worker_` is a **member** and `Panel` is its logical owner on the GUI thread; also `Worker` has an implicit parent? Members cannot be moved if they have a parent; a non-parented member *can* be moved, but then **`~Panel` destroys `worker_` on the GUI thread** while it may still be in use on `thread_`.  
4. **`thread_.terminate()`** — undefined, skips destructors. Use `quit()` + `wait()`. Destruction order: stop the thread **before** `worker_` is destroyed (`~Panel` body, or declare thread after worker carefully + wait).  
5. **`onGo` calls `worker_.compute()` directly** — runs on the GUI thread, ignoring affinity.  
6. **`label_` is never defined.**  
7. **`worker_.result()`** from the GUI without synchronization — data race if compute were actually async.  
8. No `Q_OBJECT` / slots macro setup if `SLOT()` is used.  
9. `connect` to `onGo(int)` but `onGo()` takes no `int`.

### Exercise 51

```cpp
class Stopwatch : public QObject {
    Q_OBJECT
    Q_PROPERTY(int ms READ ms NOTIFY msChanged)
    int m_ms = 0;
    QTimer* m_timer; // parented
public:
    explicit Stopwatch(QObject* parent = nullptr)
        : QObject(parent), m_timer(new QTimer(this))
    {
        m_timer->setInterval(10);
        connect(m_timer, &QTimer::timeout, this, [this] {
            setMs(m_ms + 10);
        });
    }
    int ms() const { return m_ms; }

public slots:
    void start() { m_timer->start(); }
    void stop()  { m_timer->stop(); }
    void reset() { stop(); setMs(0); }

signals:
    void msChanged(int);

private:
    void setMs(int v) {
        if (m_ms == v) return;
        m_ms = v;
        emit msChanged(m_ms);
    }
};
```

### Exercise 52

```cpp
label->setText(tr("Hello %1!").arg(name));
```

One translatable phrase; `%1` is the placeholder. Do not concatenate `tr()` fragments.
