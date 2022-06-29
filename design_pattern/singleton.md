# Design Pattern - Singleton

```mermaid
classDiagram
class Singleton {
    +Singleton Instance
    -object _syncRoot
    +Singleton()
    +Execute()
}

Singleton "*" ..> "1" Singleton
```