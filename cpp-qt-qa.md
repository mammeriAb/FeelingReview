# C++ / Qt — Questions and Answers

## C++ Core

**Q1. What is the difference between a pointer and a reference?**  
A pointer is a variable that stores an address; it can be null, reassigned, and arithmetic can be applied. A reference is an alias for an existing object; it must be initialized, cannot be null, and cannot be reseated.

**Q2. What is the difference between stack and heap allocation?**  
Stack objects have automatic lifetime tied to the scope; allocation is fast. Heap objects are allocated with `new`/`malloc` (or allocators) and must be released explicitly or via RAII. Heap is larger and slower, and mismanagement causes leaks and use-after-free.

**Q3. What does `const` mean on a member function?**  
The function promises not to modify the object’s non-`mutable` members. A `const` object (or `const` reference) can only call `const` member functions.

**Q4. What is the difference between `const T*`, `T* const`, and `const T* const`?**  
- `const T*`: pointer to const `T` (data is const).  
- `T* const`: const pointer (pointer cannot change).  
- `const T* const`: both pointer and data are const.

**Q5. What is RAII?**  
Resource Acquisition Is Initialization: a resource is acquired in a constructor and released in the destructor. When the object goes out of scope, the resource is freed even if an exception is thrown. Smart pointers, locks, and file wrappers are typical RAII types.

**Q6. What are the differences among `new`, `new[]`, `malloc`, and placement `new`?**  
`new`/`new[]` call constructors (and `new[]` for arrays). `malloc` allocates raw bytes and does not call constructors. Placement `new` constructs an object in already-allocated memory.

**Q7. What is the Rule of Three / Five / Zero?**  
If you define one of destructor, copy constructor, or copy assignment, you usually need all three (Rule of Three). In C++11, move constructor and move assignment join that set (Rule of Five). Prefer Rule of Zero: let members (smart pointers, containers) manage resources so you write none of those special members.

**Q8. What is the difference between copy and move?**  
Copy duplicates the resource. Move transfers ownership, typically leaving the source in a valid but unspecified (often empty) state. Move is used for temporaries and when copying is expensive or not allowed.

**Q9. What is `std::move`?**  
It is a cast to an rvalue reference (`T&&`). It does not move anything by itself; it enables a move constructor or move assignment to be selected.

**Q10. What is the difference between lvalue and rvalue?**  
An lvalue has an identity (you can take its address). An rvalue is a temporary or a value about to expire. Rvalue references (`T&&`) bind to rvalues and enable move semantics.

**Q11. What is a dangling pointer / dangling reference?**  
It points to (or aliases) memory that has already been freed or gone out of scope. Using it is undefined behavior.

**Q12. What is undefined behavior? Give common examples.**  
The language does not define the result. Common cases: use-after-free, double free, out-of-bounds access, signed integer overflow, uninitialized reads, data races, violating `const` via `const_cast` when the object is truly const.

**Q13. `delete this` — is it legal?**  
It can be legal if the object was allocated with `new`, no member is used after `delete this`, and ownership is clear. It is fragile and usually avoided.

**Q14. What is the difference between `struct` and `class`?**  
Default access and default inheritance are `public` for `struct` and `private` for `class`. Otherwise they are the same.

**Q15. What is a virtual function? Why is a virtual destructor needed?**  
A virtual function is dispatched through the object’s vtable, so the most-derived override is called via a base pointer/reference. If you delete a derived object through a base pointer, the destructor must be virtual; otherwise only the base destructor runs (undefined behavior / resource leak).

**Q16. What is a pure virtual function? What is an abstract class?**  
`virtual void f() = 0;` makes the class abstract: it cannot be instantiated. A derived class must override all pure virtuals (or remain abstract). You can still provide a body for a pure virtual destructor.

**Q17. What is the difference between overload, override, and hide?**  
- Overload: same name, different signatures, same scope.  
- Override: derived virtual function with the same signature as the base. Use `override` to catch mistakes.  
- Hide: a derived function with the same name hides base overloads, even with different signatures.

**Q18. What do `override` and `final` do?**  
`override` asks the compiler to check that the function actually overrides a virtual. `final` on a function forbids further overrides; on a class forbids further derivation.

**Q19. What is slicing?**  
Assigning or passing a derived object by value as a base object copies only the base part. Virtual behavior and derived members are lost.

**Q20. Multiple inheritance and the diamond problem — what is `virtual` inheritance?**  
If two bases share a common ancestor, the most-derived object can contain two copies of that ancestor. Virtual inheritance shares a single base subobject.

**Q21. What is `explicit` on a constructor or conversion operator?**  
It blocks implicit conversions. Useful for single-argument constructors (and converting operators) so you do not accidentally construct or convert.

**Q22. What are `static`, `extern`, and anonymous-namespace symbols?**  
`static` at file scope gives internal linkage. `extern` declares a symbol defined elsewhere. An anonymous namespace is the modern way to give internal linkage to a group of names.

**Q23. `inline` vs `static` vs templates — where can you define functions in headers?**  
ODR (One Definition Rule) requires one definition across the program, except for `inline` functions, function templates, and class members defined in the class body (which are implicitly inline). `static` in a header creates a separate copy per translation unit.

**Q24. What is a friend?**  
A friend function or class can access private/protected members. Friendship is not inherited, not symmetric, and not transitive.

**Q25. What is the difference between `nullptr`, `NULL`, and `0`?**  
`nullptr` has type `std::nullptr_t` and does not convert to integer types, so it is safer in overload resolution. `NULL`/`0` can bind to `int` overloads.

---

## Modern C++ (11–20)

**Q26. What are smart pointers? When do you use each?**  
- `unique_ptr`: exclusive ownership, movable, not copyable. Default choice.  
- `shared_ptr`: shared ownership via control block / refcount. Use when lifetime is genuinely shared.  
- `weak_ptr`: non-owning observer of a `shared_ptr`; breaks cycles.  
Prefer `make_unique` / `make_shared`.

**Q27. Why can `shared_ptr` cycles leak?**  
Two objects holding `shared_ptr`s to each other never reach refcount 0. Store back-references as `weak_ptr`.

**Q28. What is a lambda? What is the difference between `[=]`, `[&]`, and init-captures?**  
A lambda is an anonymous function object. `[=]` captures copies; `[&]` captures by reference (dangerous if the lambda outlives the locals). Init-capture (`[p = std::move(ptr)]`) allows move-capture and renaming.

**Q29. What is `auto`? What is `decltype`?**  
`auto` deduces the type from the initializer (dropping references and top-level `const` unless you write `auto&` / `const auto&`). `decltype(expr)` yields the declared type of the expression, preserving references.

**Q30. Range-based for vs iterators vs indices?**  
Range-for is the default for whole-container traversal. Iterators are needed for insertion/erasure and algorithms. Indices are useful for random-access containers when the position matters. Use `std::as_const` or `const` reference to avoid accidental copies.

**Q31. What are `constexpr` and `consteval`?**  
`constexpr` functions *may* be evaluated at compile time when arguments are constant. `consteval` (C++20) *must* be evaluated at compile time. `constexpr` variables are compile-time constants.

**Q32. What is `noexcept`? Why does it matter for move constructors?**  
It promises the function does not throw. `std::vector` and others only move on reallocation if the move constructor is `noexcept`; otherwise they copy to keep the strong exception guarantee.

**Q33. What are `std::optional`, `std::variant`, and `std::any`?**  
- `optional<T>`: a `T` or empty.  
- `variant<Ts...>`: one of a fixed set of types.  
- `any`: a type-erased value of any copy-constructible type (heavier).

**Q34. What is structured binding?**  
`auto [a, b] = pair;` unpacks arrays, pair/tuple-like types, and aggregates into named variables (C++17).

**Q35. What is `if constexpr`?**  
A compile-time branch. The discarded branch is not instantiated, which is essential in templates.

**Q36. What are concepts (C++20)?**  
Named constraints on template parameters. They give clearer errors and overload sets than SFINAE.

**Q37. What is the difference between `std::thread`, `std::async`, and `std::jthread`?**  
`thread` is a raw OS thread; you must `join` or `detach`. `async` may run deferred or on a thread and returns a `future`. `jthread` (C++20) joins in the destructor and supports stop tokens.

**Q38. What is a data race?**  
Two threads access the same memory, at least one write, with no synchronization. That is undefined behavior. Use mutexes, atomics, or don’t share.

**Q39. Mutex vs atomic vs lock-free?**  
Mutexes serialize critical sections. Atomics provide individual operations with memory-order guarantees. Lock-free algorithms avoid locks but are harder to get right. Default to a mutex unless profiling says otherwise.

**Q40. What does `std::lock_guard` / `unique_lock` / `scoped_lock` give you?**  
RAII locking. `lock_guard` is simplest. `unique_lock` can defer/unlock and is used with condition variables. `scoped_lock` locks multiple mutexes without deadlock (C++17).

---

## Templates, STL, and Types

**Q41. What is a template? Function template vs class template vs alias template?**  
A blueprint the compiler instantiates for types/values. Function templates deduce arguments. Class templates need explicit arguments (unless CTAD). Alias templates name a family of types (`template<class T> using Ptr = T*;`).

**Q42. What is SFINAE?**  
Substitution Failure Is Not An Error: a failed substitution discards an overload instead of erroring. Used to constrain templates (largely replaced by concepts).

**Q43. `vector` vs `list` vs `deque` vs `array` vs `string`?**  
- `vector`: contiguous, fast iteration, cheap append, expensive insert in the middle.  
- `list`: node-based, stable iterators, slow traversal. Rarely the right default.  
- `deque`: block-based, cheap insert at both ends.  
- `array`: fixed-size, stack-friendly.  
- `string`: contiguous characters, SSO for short strings.

**Q44. When does `vector` reallocation invalidate iterators?**  
When growth exceeds capacity. `reserve` avoids it. Insert/erase also invalidate at/after the point (and all on reallocation).

**Q45. `map` vs `unordered_map`?**  
`map` is an ordered tree (`O(log n)`), stable order. `unordered_map` is a hash table (`O(1)` average) requiring a hash and equality. Use unordered unless you need order or deterministic iteration.

**Q46. Why pass `const T&` instead of `T`? When pass by value?**  
`const T&` avoids copies for large or non-cheap types. Pass small cheap types (`int`, pointers, `string_view`) by value. For sinks that store the argument, pass by value and `move` in (or use forwarding references).

**Q47. What is a forwarding reference (`T&&` in a template) vs an rvalue reference?**  
`template<class T> void f(T&&)` is a forwarding (universal) reference: `T` can be `U` or `U&`. Combined with `std::forward<T>`, it preserves value category (perfect forwarding). `void f(int&&)` is only an rvalue reference.

**Q48. What is `std::function` vs a template parameter / lambda?**  
`std::function` is a type-erased callable (can store lambdas, function pointers, bind expressions) with some overhead. A template parameter is zero-overhead but the type is unique per lambda.

**Q49. How do exceptions work? What is the strong guarantee?**  
Throwing unwinds the stack, running destructors (RAII). Catch by `const` reference. The strong guarantee: the operation either succeeds or leaves state unchanged. The basic guarantee: no leaks, state remains valid. `noexcept` functions must not throw.

**Q50. Should C++ code use exceptions or error codes?**  
Be consistent. Qt largely uses return values / `bool` / null, not exceptions, in the public API. Mixing Qt objects with exceptions is possible but you must not throw across signal emissions into C code, and you must keep RAII so QObject trees still clean up.

---

## Qt — Meta-Object, QObject, Signals and Slots

**Q51. What is `QObject` and why is it the heart of Qt?**  
`QObject` provides the meta-object system: signals/slots, properties, parent-child ownership, events, `Q_OBJECT` introspection (`metaObject()`, `setProperty()`, `invokeMethod`). Most Qt classes derive from it.

**Q52. What does the `Q_OBJECT` macro do? What is `moc`?**  
`Q_OBJECT` marks the class for the Meta-Object Compiler. `moc` generates extra C++ that implements `qt_metacall`, signal implementations, and meta-data. Without it, signals/slots/properties/`qobject_cast` will not work correctly.

**Q53. Why must a `QObject` subclass’s `Q_OBJECT` class be in a header processed by moc?**  
`moc` typically runs on headers. Defining a `Q_OBJECT` class only in a `.cpp` requires `#include "file.moc"` or a moc run on the cpp. Forgetting this yields linker errors for `vtable` / `metaObject`.

**Q54. What is the parent-child tree?**  
When you pass a parent to a `QObject`, the parent deletes its children in its destructor. This is Qt’s primary ownership model for QObjects. Parents and children must live on the same thread.

**Q55. Can you copy a `QObject`?**  
No. Copy constructor and assignment are disabled. QObjects have identity (parent, connections, thread affinity). Hold them by pointer, or move widgets via layouts, not by copy.

**Q56. `new QPushButton("OK", this)` — who deletes it?**  
The parent (`this`) deletes it. You usually do not `delete` child widgets yourself. If there is no parent, you own it (`unique_ptr` or explicit `deleteLater`).

**Q57. `delete` vs `deleteLater()` on a `QObject`?**  
`delete` destroys immediately. Unsafe if the object is on the call stack of an event/slot. `deleteLater()` posts a deferred deletion to the object’s thread event loop — the safe default when destroying from a slot.

**Q58. What are signals and slots?**  
Signals are emitted on events. Slots are functions that run in response. The connection is loosely coupled: the emitter does not need to know the receiver.

**Q59. Old `SIGNAL()`/`SLOT()` macros vs function-pointer / lambda syntax?**  
Macros are string-based, checked at runtime, and allow default arguments in the signature string. Pointer-to-member / lambda connections are checked at compile time and can connect to functors. Prefer PMF/lambda syntax.

**Q60. What connection types exist?**  
- `AutoConnection` (default): direct if same thread, queued if different.  
- `DirectConnection`: call immediately in the emitter’s thread.  
- `QueuedConnection`: post an event; slot runs in the receiver’s thread.  
- `BlockingQueuedConnection`: like queued but the emitter waits (deadlock if same thread).  
- `UniqueConnection`: refuse duplicate connections.

**Q61. What can you connect? Signature rules?**  
The slot/functor must accept arguments that can be created from the signal’s arguments (same types or fewer trailing args). Extra slot parameters are not allowed. Queued connections require registered metatypes for the argument types.

**Q62. Why `qRegisterMetaType<T>()` / `Q_DECLARE_METATYPE`?**  
Queued connections and `QVariant` need a way to copy arguments. For custom types, declare with `Q_DECLARE_METATYPE` and, for queued queued-across-threads, register with `qRegisterMetaType<T>()`.

**Q63. Can a signal connect to another signal?**  
Yes. Emitting the first emits the second. Useful for forwarding.

**Q64. What does `emit` do?**  
It is an empty macro. `emit valueChanged(v);` is just a function call to the moc-generated signal. It exists for readability.

**Q65. How do you disconnect?**  
`QObject::disconnect(...)`, or hold a `QMetaObject::Connection` and `disconnect(conn)`. Connections are removed automatically when sender or receiver is destroyed.

**Q66. What is a queued slot invocation from another thread (`QMetaObject::invokeMethod`)?**  
`invokeMethod(obj, "foo", Qt::QueuedConnection, ...)` or the PMF overload posts a call onto `obj`’s thread. This is the safe way to talk to a `QObject` living in another thread.

**Q67. What is `qobject_cast` vs `dynamic_cast`?**  
`qobject_cast<T*>(obj)` uses the meta-object; it does not require RTTI and works across plugin boundaries better. It only works for `QObject` types with `Q_OBJECT`. `dynamic_cast` needs RTTI and works for any polymorphic type.

**Q68. What are Qt properties (`Q_PROPERTY`)?**  
A named, introspectable value with READ/WRITE/NOTIFY. Used by QML, Designer, animations, and `setProperty`. `NOTIFY` should be a signal emitted when the value changes.

**Q69. What is `QVariant`?**  
A type-erased value that can hold Qt types and registered custom types. Used in models, properties, settings, and SQL. Prefer concrete types in C++ APIs; use `QVariant` at dynamic boundaries.

**Q70. `QString` vs `std::string` vs `QByteArray` vs `QLatin1String`?**  
`QString` is UTF-16 (historically; Qt 6 still uses UTF-16 storage). `QByteArray` is bytes (UTF-8 data, binary). `std::string` is not UTF-aware. `QLatin1String`/`QStringView`/`QUtf8StringView` avoid allocations when passing literals. Convert with `toUtf8()`, `fromUtf8()`, `toStdString()` carefully.

---

## Qt Event Loop, Widgets, and UI

**Q71. What is the event loop?**  
`QCoreApplication::exec()` processes events (input, timers, sockets, queued signals, deferred deletes). Qt is event-driven: almost everything interesting happens as an event or a slot.

**Q72. `QApplication` vs `QGuiApplication` vs `QCoreApplication`?**  
- `QCoreApplication`: no GUI (services, console).  
- `QGuiApplication`: Qt Gui + windows, no Widgets.  
- `QApplication`: Widgets (includes Gui). You need one, created first, destroyed last.

**Q73. What is an event vs a signal?**  
Events (`QEvent`) are dispatched to objects via `event()` / `eventFilter()`. Signals are the meta-object observer mechanism. Input and lifecycle tend to be events; high-level “something happened” APIs tend to be signals. You can convert between them.

**Q74. How do you handle a key press on a widget?**  
Subclass and override `keyPressEvent`, or install an event filter, or use `QShortcut` / `QAction`. Call the base class for unhandled keys.

**Q75. What is an event filter?**  
`obj->installEventFilter(filter)` lets `filter->eventFilter(obj, event)` see events before the target. Return `true` to consume. Useful without subclassing. Filters run in the target’s thread.

**Q76. `accept()` vs `ignore()` on `QEvent` / `QCloseEvent`?**  
`accept()` means you handled it (e.g. close the window). `ignore()` on close cancels closing. For most input events, ignoring lets the parent see it.

**Q77. What are layouts (`QVBoxLayout`, `QGridLayout`, …)?**  
They size and position child widgets. Set a layout on a widget; adding a widget to a layout reparents it. Do not mix manual `move`/`setGeometry` with layouts on the same children.

**Q78. `QWidget` vs `QWindow` vs `QQuickWindow`?**  
`QWidget` is the widgets-system control. `QWindow` is a native window from Qt Gui. `QQuickWindow`/`QQuickView` host a Qt Quick scene. You can embed Quick in widgets via `QQuickWidget` (with caveats).

**Q79. What is a `QAction`?**  
An abstract command (text, icon, shortcut, checkable) that can be placed on menus, toolbars, and buttons. One action, many UI surfaces.

**Q80. Modal vs modeless dialogs. `exec()` vs `show()` vs `open()`?**  
`exec()` runs a nested event loop and blocks the caller (modal). `open()` is window-modal without a nested loop (preferred). `show()` is modeless. Nested `exec()` event loops can re-enter your code; be careful with lifetime.

**Q81. How do you show a message / file dialog?**  
`QMessageBox::information/question/...` or `QFileDialog::getOpenFileName`. These static helpers `exec()` internally. For production UIs, consider non-static, non-blocking dialogs.

**Q82. What is `QTimer`? Zero-timeout timers?**  
`QTimer` fires `timeout` on the owner’s thread via the event loop. `interval 0` means “run when the event loop is idle” — useful to defer work without a thread. Use `singleShot` for one-shot. Use `Qt::PreciseTimer` when needed.

**Q83. Why must GUI objects stay on the main thread?**  
Qt Widgets and Quick item trees are not thread-safe. Only the GUI thread may create/touch widgets. Worker threads compute, then marshal results with queued connections.

**Q84. `update()` vs `repaint()` vs `QPainter` in `paintEvent`?**  
`update()` schedules a paint (coalesced). `repaint()` paints immediately (usually avoid). Drawing happens in `paintEvent` with `QPainter`. Painting outside `paintEvent` is unsupported for widgets.

**Q85. What is the difference between `show()`, `hide()`, `setVisible()`, and `close()`?**  
`show`/`hide`/`setVisible` change visibility. `close()` sends `QCloseEvent`; if accepted, the widget hides and, if `Qt::WA_DeleteOnClose` is set, is deleted.

**Q86. Size hints: `sizeHint()`, `minimumSizeHint()`, stretch, and size policies?**  
Layouts query size hints and `QSizePolicy` (Fixed, Preferred, Expanding, …) plus stretch factors to distribute extra space. Override `sizeHint()` on custom widgets.

---

## Model/View, Data, I/O

**Q87. What is Qt’s Model/View architecture?**  
Data lives in a model (`QAbstractItemModel`). Views (`QListView`, `QTableView`, `QTreeView`) display it. Delegates render and edit cells. Multiple views can share one model.

**Q88. `QStandardItemModel` vs custom `QAbstractTableModel`?**  
`QStandardItemModel` is convenient and item-based. For large or live data, implement `QAbstractTableModel`/`List`/`Tree` and emit `dataChanged` / `beginInsertRows` correctly. Never mutate model internals without the begin/end signals.

**Q89. What roles does `data()` use?**  
`Qt::DisplayRole`, `EditRole`, `DecorationRole`, `ToolTipRole`, `UserRole` and up. Views ask for roles; your `data()` returns `QVariant{}` for unsupported ones.

**Q90. What is `QItemSelectionModel`?**  
Holds selection/current index independently of the view so two views can share selection, or you can drive selection from code.

**Q91. `QFile`, `QDataStream`, `QTextStream`, `QSettings`?**  
`QFile` is I/O. `QDataStream` is binary Qt serialization. `QTextStream` is text with codec. `QSettings` is persistent key/value (registry/plist/ini).

**Q92. How do you run a network request in Qt?**  
`QNetworkAccessManager` + `QNetworkReply`. Asynchronous: connect to `finished`. Do not use `QEventLoop` locally to fake sync unless you fully understand reentrancy. Handle SSL errors and `deleteLater` the reply.

---

## Threading in Qt

**Q93. How do you use worker threads correctly with QObjects?**  
Two patterns:  
1. `QThread` subclass with `run()` for non-QObject work, emit signals to the GUI.  
2. Worker `QObject` moved with `moveToThread(thread)`: start work via queued signal, never call worker methods directly from the GUI thread.

**Q94. What does `moveToThread` actually change?**  
The object’s *thread affinity*: where its slots, timers, and events run. You can only `moveToThread` an object with no parent, and you should move it from its current thread. Children move with the parent.

**Q95. Why is `QThread::run()` calling GUI APIs wrong?**  
`run()` executes on the worker thread. Widgets and many Qt Gui APIs are main-thread only. Also: subclassing `QThread` and putting slots on the `QThread` object is a common trap — the `QThread` *object* lives in the thread that created it, not in `run()`’s thread.

**Q96. `QtConcurrent` vs `QThreadPool` vs `QThread`?**  
`QtConcurrent::run` / `mapped` uses the global thread pool for fire-and-forget or `QFuture`. `QThreadPool` + `QRunnable` is similar, lower level. Use a dedicated `QThread` when you need a lasting event loop (sockets, worker QObject).

**Q97. Are Qt containers thread-safe?**  
No, except documented cases (`QMutex`, atomics, some implicitly shared copies as *separate* objects). Implicit sharing is copy-on-write: sharing a container across threads without synchronization is a data race if one side detaches/writes.

**Q98. How do you stop a worker cleanly?**  
Set an atomic/flag or use a queued `stop()` slot, then `thread->quit()` and `wait()`. Do not `terminate()`. For `QFuture`, use `QFutureWatcher` and cooperative cancel.

---

## QML / Qt Quick

**Q99. What is QML vs Qt Quick vs Widgets?**  
QML is the language. Qt Quick is the scene-graph UI toolkit (items, animations). Widgets are the classic C++ CPU-rasterized controls. You can mix, but pick one as the primary UI.

**Q100. How do you expose C++ to QML?**  
- `qmlRegisterType<T>("Pkg", 1, 0, "MyItem")` or `QML_ELEMENT`.  
- Set a context property (`engine.rootContext()->setContextProperty`) for a singleton-like object.  
- `QQmlEngine::setObjectOwnership` to control deletion. Properties, slots, `Q_INVOKABLE`, and signals are visible if they go through the meta-object system.

**Q101. What is a `Q_INVOKABLE` vs a slot in QML?**  
Both are callable from QML. Slots can also be connected to signals. Properties with `NOTIFY` are the preferred way to expose state.

**Q102. Why must you emit the `NOTIFY` signal when a property changes?**  
QML bindings re-evaluate when that signal fires. Forgetting it leaves the UI stale. Don’t emit if the value did not change (avoid binding loops).

**Q103. QML ownership: who deletes a C++ object passed to QML?**  
By default, objects created in C++ and set as context properties are C++-owned. Objects created in QML are JS/QML-owned. Returning a `QObject*` from an invokable often transfers ownership to QML unless you set `CppOwnership`. Double-free is a common bug.

**Q104. List models: `QAbstractListModel` in QML vs `ListModel`?**  
For real data, implement `QAbstractListModel` in C++ (`roleNames()`, `data()`, insert/remove). `ListModel` is fine for small QML-only data. Always emit the right model signals.

**Q105. What is the Qt Quick scene graph / why not block the GUI thread?**  
Items are rendered on the render thread, but JS/QML bindings and C++ slots still run on the GUI thread. Heavy work in a click handler janks animations. Offload to workers; update via properties/signals.

---

## Memory, Debugging, Build

**Q106. Qt parent-child vs C++ smart pointers — can you mix them?**  
Do not manage the same QObject with both a parent and `unique_ptr` (double delete). Common pattern: parented widgets use Qt ownership; non-parented QObjects can be `unique_ptr<QObject, Deleter>` that calls `deleteLater`, or a parented child of a controller object.

**Q107. What is implicit sharing (copy-on-write) in Qt?**  
Types like `QString`, `QImage`, `QList` (Qt5) share a payload until a non-const operation detaches. Passing by value is cheap. Detach on write can surprise you in multithreaded use and with `QList<T>::operator[]` non-const.

**Q108. `QList` vs `QVector` vs `std::vector` (Qt 5 vs Qt 6)?**  
In Qt 5, `QList` was an array of pointers for large T; `QVector` was contiguous. In Qt 6, `QList` is contiguous (alias-like with `QVector`). For generic C++, `std::vector` is fine; for Qt APIs, use the Qt containers they expect.

**Q109. What is `QPointer` / `QWeakPointer` / `QSharedPointer`?**  
`QPointer<T>` is a guarded pointer: it becomes null when the QObject is destroyed (does not own). `QSharedPointer`/`QWeakPointer` are Qt’s shared/weak pointers; you can mix with `QObject` via `QSharedPointer<T>(obj, &QObject::deleteLater)` but parent-child plus shared_ptr is easy to get wrong.

**Q110. How do you debug “QObject::setParent: Cannot set parent, new parent is in a different thread”?**  
You created or reparented across threads. Create QObjects in the thread they will live in, or `moveToThread` before setting a parent. Don’t create widgets off the main thread.

**Q111. “Cannot create children for a parent that is in a different thread” — same family of bugs.**  
Timers, sockets, and children inherit affinity. Create them in the worker after `moveToThread`, not in the worker’s constructor (constructor still runs in the creating thread).

**Q112. Typical moc / linker errors (`undefined reference to vtable for MyClass`)?**  
Missing `Q_OBJECT`, class not moc’d, forgot to rebuild, or a virtual function not defined. Clean and rerun qmake/CMake.

**Q113. qmake vs CMake for Qt?**  
qmake (`*.pro`) is the older Qt-native build. CMake + `qt_add_executable` / `Qt6::Widgets` is current upstream recommendation. Both must run moc, uic, rcc (AUTOMOC/AUTOUIC/AUTORCC).

**Q114. What are `uic` and `rcc`?**  
`uic` compiles `.ui` Designer files to `ui_*.h`. `rcc` embeds `.qrc` resources (icons, QML, translations) into the binary. Access with `":/path/in/qrc"`.

**Q115. How do you localize a Qt app?**  
Mark strings with `tr()` in QObject subclasses (`QCoreApplication::translate` otherwise). `lupdate` → `.ts` → linguist → `lrelease` → `.qm`. Load with `QTranslator`. In QML use `qsTr()`.

**Q116. `qDebug()`, categories, and `QLoggingCategory`?**  
`qDebug()`, `qInfo()`, `qWarning()`, `qCritical()`, `qFatal()`. Use `QLoggingCategory` to turn areas on/off. Don’t log secrets. Prefer streaming: `qDebug() << "x" << x;`.

**Q117. How do you write a unit test with Qt?**  
`QTest` (`Qt Test`): `QCOMPARE`, `QVERIFY`, `QSignalSpy`, data-driven tests with `_data()`. GUI tests can use `QTest::mouseClick`. Keep tests headless with `QT_QPA_PLATFORM=offscreen` where possible.

**Q118. What is the pimpl idiom in Qt (`d-pointer`)?**  
Private implementation behind a `QScopedPointer<ClassPrivate> d`. Keeps the public header binary-stable (BIC) and compiles faster. Qt uses `Q_DECLARE_PRIVATE` / `Q_D`.

---

## Mixed C++ / Qt Design

**Q119. When should a class *not* inherit `QObject`?**  
Value types (`QString`-like), POD data, and anything you want to copy. Inheritance from `QObject` is for identity, signals, properties, and parented lifetime. Prefer composition: a small QObject adapter around a plain C++ model.

**Q120. How do you keep business logic testable and UI-agnostic?**  
Put logic in plain C++ or QObject models with signals. UI (widgets/QML) binds to properties and calls slots. Avoid putting domain rules inside `paintEvent` or QML `onClicked` beyond forwarding.

**Q121. Signal/slot vs callback vs observer — why Qt’s approach?**  
Connections are introspectable, thread-marshaled, and auto-disconnected on destroy. Cost: moc, a bit of runtime, and QObject identity. For tight inner loops, use direct calls or `std::function`.

**Q122. How do you pass a custom struct through a queued connection?**  
Make it copyable, `Q_DECLARE_METATYPE(MyType)`, call `qRegisterMetaType<MyType>()` at startup, emit by value. For large data, pass `QSharedPointer<const T>` or an id and keep the payload in a model.

**Q123. What happens if a slot throws an exception?**  
Qt does not use exceptions in its own code. An exception escaping a slot can `std::terminate` or skip remaining connections depending on version/compiler. Catch inside slots.

**Q124. How do you debounce a signal (e.g. textChanged search)?**  
`QTimer` single-shot restarted on each signal; when it fires, run the query. In QML, `Timer`. Do not start a thread per keystroke.

**Q125. Widget painting is slow — what do you check?**  
Paint only in `paintEvent`, use `update(rect)`, cache with `QPixmap`/`QImage`, avoid allocating in the paint path, use `QStyle` primitives, consider scene-graph (Quick) for huge item counts, or a model/view instead of thousands of widgets.

**Q126. Thousands of `QWidget` children vs a view?**  
Widgets are heavy (each is a QObject + windowing state). `QTableView`/`QListView` create delegates for visible cells only. For custom canvas, `QGraphicsView` or Qt Quick.

**Q127. `QGraphicsView` vs Qt Quick vs Widgets — when?**  
Widgets: desktop forms, native look, tools. Graphics View: large 2D item scenes in C++. Qt Quick: fluid, animated, touch, declarative UI. Don’t stack all three without a clear boundary.

**Q128. How do you handle high-DPI?**  
Qt 6 is high-DPI aware by default. Use vector icons (`QIcon` with SVG), layouts instead of hardcoded pixels, `devicePixelRatio` for custom images. Avoid assuming 96 DPI.

**Q129. What is `QEventLoop` reentrancy and why is it dangerous?**  
`QDialog::exec()`, `QMessageBox`, nested loops, and `QEventLoop::exec()` process events while your function is on the stack. Users can click other buttons, destroy objects, or re-enter the same slot. Prefer `open()`, queued follow-ups, and `deleteLater`.

**Q130. Checklist for a Qt slot that starts async work?**  
Disable or guard the UI, don’t capture `this` in a lambda without `QPointer`/`QObject*` guard, connect `finished` with `Qt::SingleShotConnection` or disconnect, always `deleteLater` replies, marshal back to the GUI thread, handle destruction of the widget mid-request.

---

## Quick fire

**Q131. `this` captured in a lambda connected to a signal — is it safe?**  
Only if the connection is to `this` as context (`connect(sender, &X::sig, this, [this]{...})`). The context object disconnects the functor when destroyed. Without a context, the lambda can run after `this` is dead.

**Q132. Default arguments in signals/slots with PMF syntax?**  
The PMF connection does not use default arguments. Connect to a lambda that supplies them, or keep a trampoline slot.

**Q133. Can signals be virtual? Can slots be virtual?**  
Signals should not be virtual (moc implements them). Slots can be virtual; the actual callee is the dynamic type when invoked as a normal C++ call. Queued invocations still call the meta-object method that was connected.

**Q134. `const` signals / slots?**  
Slots can be `const`. Signals are not const in practice (emitting mutates connection lists). Connecting to const objects works for const slots.

**Q135. What is `Q_GADGET`?**  
A lightweight meta-object without signals/slots or QObject identity. Good for value types that need properties/enums in QML (`Q_PROPERTY`, `Q_ENUM`).

**Q136. `Q_ENUM` vs `Q_ENUM_NS` vs `Q_FLAG`?**  
Registers enumerations with the meta-object so QML and `QVariant` can use names. `Q_FLAG` is for OR-able flags. `Q_ENUM_NS` is for enums in a namespace with `Q_NAMESPACE`.

**Q137. Why `tr()` and not `tr("Open %1").arg(name)` issues?**  
`tr()` must see a string literal (or disambiguation). Build the format inside `tr("%1 items").arg(n)`. Translators need the full phrase, not concatenated fragments.

**Q138. `QStringBuilder` / `%` / `arg` vs `+`?**  
Repeated `+` may allocate intermediates. `QStringLiteral`/`u"..."` and `arg`/`QStringBuilder` (`%`) are faster. Prefer `QStringLiteral` for constants.

**Q139. Include what you use: `QtWidgets` module vs forwarding?**  
Forward-declare `QClass` in headers when you only need pointers/references. Include the module headers in the `.cpp`. Faster builds, less coupling.

**Q140. What does `QCoreApplication::processEvents()` do and why is it usually wrong?**  
It pumps the event loop in the middle of your function (reentrancy, like a mini `exec()`). Prefer `QTimer::singleShot(0, ...)` or a worker thread with progress signals.

---

## Mini scenarios

**Q141. Button click should load a file without freezing the UI. Outline the design.**  
On click, disable the button, start `QtConcurrent::run` or a worker object to read/parse. When done, emit a signal with the result (registered type or `shared_ptr`). GUI slot updates the model/view and re-enables the button. Errors via a separate signal. Guard with `QPointer<MyWidget>` if using lambdas.

**Q142. You `moveToThread` a worker, then call `worker->doWork()` from the GUI. Why is that wrong?**  
That is a direct call on the GUI thread, ignoring affinity. Emit a signal (queued) or `invokeMethod` with `QueuedConnection`. Direct calls are only OK for thread-safe, non-QObject functions.

**Q143. Model `data()` is called from a view on the GUI thread, but data lives in a worker. What now?**  
The model must be GUI-thread. Worker posts snapshots or incremental `beginInsertRows`/`dataChanged` via queued signals. Don’t lock inside `data()` waiting on the worker (jank/deadlock).

**Q144. Double-delete crash after closing a dialog that `new`’d a widget with parent `this` and also stored `unique_ptr`.**  
Pick one owner. If parented, don’t use `unique_ptr`. If `unique_ptr`, pass `nullptr` parent.

**Q145. QML binding loop detected.**  
Two properties update each other via bindings/NOTIFY. Break the cycle: write to the model in a handler (`onClicked`) instead of a two-way binding, or guard setters so they no-op when the value is unchanged.

---

## Cheat sheet

| Topic | Rule of thumb |
|---|---|
| QObject lifetime | Parent deletes children; otherwise `deleteLater` |
| Threads | GUI objects on GUI thread; talk via queued signals |
| Connections | PMF/lambda + context object `this` |
| Models | Always `begin*`/`end*` around structural changes |
| Copy QObject | Never; disable copy, use pointers |
| Resources | RAII in C++; parent tree in Qt |
| Heavy work | Worker / `QtConcurrent`, not `processEvents` |
| QML C++ API | Properties + NOTIFY + `Q_INVOKABLE`/slots |
| Build | CMake + AUTOMOC; missing moc → vtable errors |
| Exceptions | Catch in slots; Qt APIs are mostly non-throwing |
