import json
import time
import sys
import os
from deep_translator import GoogleTranslator

# ---------- НАСТРОЙКИ ----------
EN_FILE = "EN.json"
PROGRESS_FILE = "RU_progress.json"      # промежуточный файл
FINAL_FILE = "RU_crazy_full.json"       # итоговый файл

CHAIN = CHAIN = ['en', 'ru', 'ja', 'ko', 'th', 'ar', 'iw', 'sw', 'zh-CN', 'de', 'fr', 'it', 'en', 'ru'] # цепочка (можно менять)
SLEEP_TIME = 0.7
TIMEOUT = 5                             # таймаут запроса
# -------------------------------

def load_json(path):
    try:
        with open(path, 'r', encoding='utf-8') as f:
            return json.load(f)
    except (FileNotFoundError, json.JSONDecodeError):
        return None

def save_json(obj, path):
    with open(path, 'w', encoding='utf-8') as f:
        json.dump(obj, f, ensure_ascii=False, indent=2)

def mad_translate(text):
    """Перевод по цепочке. При любых ошибках возвращает оригинал."""
    current = text
    for i in range(len(CHAIN) - 1):
        src = CHAIN[i]
        tgt = CHAIN[i+1]
        try:
            current = GoogleTranslator(source=src, target=tgt, timeout=TIMEOUT).translate(current)
            time.sleep(SLEEP_TIME)
        except Exception as e:
            print(f"  Ошибка {src}->{tgt}: {e}")
            return text
    return current

def recursive_translate(original, translated, path=""):
    """
    Рекурсивно переводит непереведённые строки.
    original – исходные данные из EN.json
    translated – текущий прогресс (может быть неполным)
    Возвращает обновлённый translated.
    """
    if isinstance(original, dict):
        if not isinstance(translated, dict):
            translated = {}
        for key, orig_val in original.items():
            new_path = f"{path}.{key}" if path else key
            if key not in translated:
                translated[key] = None
            translated[key] = recursive_translate(orig_val, translated[key], new_path)
        return translated

    elif isinstance(original, list):
        if not isinstance(translated, list):
            translated = [None] * len(original)
        else:
            while len(translated) < len(original):
                translated.append(None)
        for i in range(len(original)):
            new_path = f"{path}[{i}]"
            translated[i] = recursive_translate(original[i], translated[i], new_path)
        return translated

    elif isinstance(original, str):
        # Если уже есть нормальный перевод – не трогаем
        if isinstance(translated, str) and translated != original and translated.strip():
            return translated
        if not original.strip():
            return original

        print(f"Перевожу: {path}")
        result = mad_translate(original)
        # Сохраняем прогресс после каждой строки
        save_json(current_data, PROGRESS_FILE)
        return result
    else:
        return original if translated is None else translated

# ---------- Загрузка ----------
print("Загружаю оригинал...")
en_data = load_json(EN_FILE)
if en_data is None:
    print(f"Ошибка: {EN_FILE} не найден.")
    sys.exit(1)

progress = load_json(PROGRESS_FILE)
if progress is None:
    print("Прогресс не найден. Начинаем с нуля.")
    # Создаём заготовку с None на месте всех строк
    def nullify(obj):
        if isinstance(obj, dict):
            return {k: nullify(v) for k, v in obj.items()}
        elif isinstance(obj, list):
            return [nullify(item) for item in obj]
        elif isinstance(obj, str):
            return None
        else:
            return obj
    progress = nullify(en_data)
else:
    print("Прогресс найден. Продолжаем...")

current_data = progress

# ---------- Главный цикл с защитой от Ctrl+C ----------
try:
    final = recursive_translate(en_data, current_data)
except KeyboardInterrupt:
    print("\n\nПеревод прерван пользователем. Сохраняю прогресс...")
    save_json(current_data, PROGRESS_FILE)
    print(f"Прогресс сохранён в {PROGRESS_FILE}. Запустите скрипт снова, чтобы продолжить.")
    sys.exit(0)
except Exception as e:
    print(f"\nНепредвиденная ошибка: {e}")
    save_json(current_data, PROGRESS_FILE)
    print(f"Прогресс сохранён. Вы можете продолжить позже.")
    sys.exit(1)

# Если дошли сюда без Ctrl+C – всё переведено
save_json(final, FINAL_FILE)
print(f"\n🎉 Полный перевод завершён! Файл: {FINAL_FILE}")
# Удаляем файл прогресса, чтобы в следующий раз начать заново (по желанию)
if os.path.exists(PROGRESS_FILE):
    os.remove(PROGRESS_FILE)
