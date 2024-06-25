---
title: "Исправление проблемы с переключением раскладки клавиатуры в Visual Studio Code"
date: 2021-04-15
weight: 10
description: >
  При переключении раскладки через Alt+Shift активируется верхнее меню и продолжение ввода текста становится невозможным из-за потери фокуса.
tags:
  - Visual Studio Code
  - IDE
---

Причина кроется в обработке нажатия клавиши 'Alt', даже в составе комбинации, что приводит к вызову меню.

Решение найдено здесь: [https://github.com/microsoft/vscode/issues/85405#issuecomment-656659452](https://github.com/microsoft/vscode/issues/85405#issuecomment-656659452).

В настройках Visual Studio Code ищем опцию 'window.titleBarStyle', которая по умолчанию установлена в 'native'. После смены значения на 'custom', нажатие клавиши 'Alt' перестаёт вызывать меню.
