# Про читы на Linux

Когда я сидел на Linux Mint, мне тоже хотелось попробовать поиграть с читами в `Counter-Strike 2`. У меня ушло примерно два дня на то, чтобы найти нормальные рабочие варианты и разобраться, как их запускать.

Из того, что я тогда нашёл:

1. [Osiris](https://github.com/danielkrupinski/Osiris)
2. [deadlocked](https://github.com/avitran0/deadlocked)

Хочу рассказать именно про первый вариант, потому что второй оказался заметно проще в запуске.

## Osiris

Чтобы запустить `Osiris`, мне потребовалось некоторое время на то, чтобы написать собственный инжектор на Python, который загружает `.so`-файл в процесс `CS2`.

Если посмотреть репозиторий чита, там предлагают более простой способ запуска:

```bash
sudo gdb -batch-silent -p "$(pidof cs2)" -ex "call (void*)dlopen(\"$PWD/libOsiris.so\", 2)"
```

Но, насколько я понял из описания автора, такой способ потенциально может палиться VAC. Поэтому я сделал свой вариант. Насколько могу судить по личному опыту, мой аккаунт после этого способа в бан не ушёл. Плюс такого подхода в том, что можно загружать не только `libOsiris.so`, но и в целом любой `.so`-файл в процесс игры.

## Мой инжектор

```python
#!/usr/bin/env python3
import os
import subprocess
import sys


def find_so_file():
    possible_paths = [
        "/home/sergey/Osiris/build/libOsiris.so",
        "/home/sergey/Osiris/build/Source/Osiris/libOsiris.so",
        "/home/sergey/Osiris/build/Source/libOsiris.so",
    ]

    for path in possible_paths:
        if os.path.exists(path):
            return path
    return None


def main():
    print("=== Simple Linux Injector ===\n")

    auto_path = find_so_file()
    if auto_path:
        print(f"✅ Автонайден файл: {auto_path}")
        use_auto = input("Использовать этот путь? [y/n]: ").lower()
        if use_auto == "y":
            lib_path = auto_path
        else:
            lib_path = input("Введи путь к .so файлу: ").strip()
    else:
        lib_path = input("Введи путь к .so файлу: ").strip()

    if not os.path.exists(lib_path):
        print("❌ Файл не найден!")
        return

    try:
        result = subprocess.run(["pidof", "cs2"], capture_output=True, text=True)
        if not result.stdout.strip():
            print("❌ CS2 не запущен!")
            return

        cs2_pid = result.stdout.strip()
        print(f"✅ CS2 найден (PID: {cs2_pid})")
    except:
        print("❌ Ошибка поиска CS2")
        return

    print("Инжект...")
    cmd = f'sudo gdb -batch-silent -p {cs2_pid} -ex "call (void*)dlopen(\\"{lib_path}\\", 2)" -ex "detach" -ex "quit"'

    try:
        subprocess.run(cmd, shell=True, check=True)
        print("✅ Инжект успешен!")
    except:
        print("❌ Ошибка инжекта!")


if __name__ == "__main__":
    main()
```

## Что было дальше

Когда мне удалось запустить чит, я примерно две недели играл с ним на разных пабликах и наблюдал за реакцией админов, когда меня вызывали на проверку. За всё это время сам софт так и не нашли.

Дополнительный момент был в том, что Linux на многих пабликах вообще запрещён, и там бан могли дать сразу только из-за самой системы. Но с некоторыми админами у меня получалось выстроить нормальный разговор: по их словам, по голосу я казался обычным игроком без читов и без попыток троллинга. Я даже просил их показать, где именно в правилах прописан запрет Linux, потому что сам читал правила и прямого запрета там не видел. Обычно они ссылались на пункты про программы, которые мешают проверке.

В любом случае отдельно мне было довольно забавно наблюдать за реакцией админов во время проверок.

## Итог

К сожалению,  HVH читов под Linux я не нашёл. Так же рекомендую запускать эти оба чита вместе, так как у одного есть то, чего у другого чита не было, и с wh есть проблема, что отображалась только половина врагов.
