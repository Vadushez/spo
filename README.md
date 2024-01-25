1. Написать командный файл печатающий N первых строк всех файлов с расширением ".c" в текущем каталоге и всех его подкаталогах. N задается как параметр в командной строке
#!/bin/bash

if [ "$#" -ne 1 ]; then
    echo "Использование: $0 <число строк>"
    exit 1
fi

N=$1

# Найти все файлы с расширением ".c" в текущем каталоге и его подкаталогах
files=$(find . -type f -name "*.c")

# Печать первых N строк каждого файла
for file in $files; do
    echo "Первые $N строк файла $file:"
    head -n $N $file
    echo "----------------------------------------"
done
====================================================================================
====================================================================================
====================================================================================
2. Написать командный файл, обеспечивающий поиск и вывод на печать имен всех файлов с одинаковым содержимым. Поиск файлов производится во всех подкаталогах заданного каталога.
#!/bin/bash

if [ "$#" -ne 1 ]; then
    echo "Использование: $0 <каталог>"
    exit 1
fi

directory=$1

# Создаем временный файл для хранения хэш-сумм
temp_file=$(mktemp)

# Находим все файлы в подкаталогах
files=$(find "$directory" -type f)

# Проходим по каждому файлу и сравниваем хэш-суммы
for file in $files; do
    md5sum "$file" >> "$temp_file"
done

# Находим дублирующиеся хэш-суммы и выводим соответствующие файлы
duplicates=$(sort "$temp_file" | uniq -d -w 32 | cut -c 35-)

echo "Файлы с одинаковым содержимым:"
while IFS= read -r dup; do
    grep "$dup" "$temp_file" | cut -c 35-
    echo "----------------------------------------"
done <<< "$duplicates"

# Удаляем временный файл
rm "$temp_file"

====================================================================================
====================================================================================
====================================================================================

3. ﻿﻿Написать командный файл, выводящий данные о числе файлов во всех
подкаталогах заданного каталога.

#!/bin/bash

if [ "$#" -ne 1 ]; then
    echo "Использование: $0 <каталог>"
    exit 1
fi

directory=$1

# Используем find для поиска всех файлов в подкаталогах каталога
# Подсчитываем число строк, чтобы получить общее количество файлов
num_files=$(find "$directory" -type f | wc -l)

echo "Общее число файлов в подкаталогах $directory: $num_files"

====================================================================================
====================================================================================
====================================================================================

4. ﻿﻿Написать командный файл, выводящий данные о совокупном объеме файлов во всех подкаталогах заданного каталога.
#!/bin/bash

if [ "$#" -ne 1 ]; then
    echo "Использование: $0 <каталог>"
    exit 1
fi

directory=$1

# Используем du для подсчета совокупного объема файлов в каталоге и его подкаталогах
total_size=$(du -sh "$directory" | cut -f1)

echo "Совокупный объем файлов в подкаталогах $directory: $total_size"

====================================================================================
====================================================================================
====================================================================================

5. ﻿﻿Написать командный файл, выводящий названия всех подкаталогов каталога /home, принадлежащих пользователям, не зарегистрированным в системе.
#!/bin/bash

# Получаем список всех пользователей из системы
all_users=$(getent passwd | cut -d: -f1)

# Получаем список подкаталогов в /home
subdirectories=$(ls -l /home | grep '^d' | awk '{print $9}')

# Фильтруем подкаталоги, принадлежащие пользователям не из списка всех пользователей
for subdir in $subdirectories; do
    owner=$(ls -ld /home/"$subdir" | awk '{print $3}')
    if ! echo "$all_users" | grep -q "$owner"; then
        echo "$subdir"
    fi
done

====================================================================================
====================================================================================
====================================================================================

6. Написать командный файл, находящий в подкаталогах заданного ката-
лога файл наибольшего обьема и выдающий путь к нему.
#!/bin/bash

if [ "$#" -ne 1 ]; then
    echo "Использование: $0 <каталог>"
    exit 1
fi

directory=$1

# Используем find для поиска всех файлов в подкаталогах каталога
# Используем du для определения объема каждого файла
# Используем sort для сортировки по объему в обратном порядке
# И выводим первую строку, которая будет файлом наибольшего объема
largest_file=$(find "$directory" -type f -exec du -h {} + | sort -rh | head -n 1)

echo "Наибольший файл:"
echo "$largest_file"

====================================================================================
====================================================================================
====================================================================================

7. ﻿﻿Написать командный файл, находящий в подкаталогах заданного каталога файлы, имеющие размер больше заданного. Вывести список владельцев этих файлов.
#!/bin/bash

if [ "$#" -ne 2 ]; then
    echo "Использование: $0 <каталог> <размер в байтах>"
    exit 1
fi

directory=$1
min_size=$2

# Используем find для поиска файлов в подкаталогах каталога с размером больше заданного
# Используем stat для получения владельца каждого файла
find "$directory" -type f -size +"$min_size"c -exec stat -c "%U %n" {} \;

====================================================================================
====================================================================================
====================================================================================

8. ﻿﻿Написать командный файл, подсчитывающий средний обьем файлов в каталогах всех пользователей системы. Вывести таблицу, в которой для каждого пользователя указать отклонение обьема его домашнего каталога от среднего объема в процентах.

#!/bin/bash

# Получаем список всех пользователей из системы
all_users=$(getent passwd | cut -d: -f1)

# Создаем временный файл для хранения данных о размерах домашних каталогов
temp_file=$(mktemp)

# Проходим по каждому пользователю и записываем размер его домашнего каталога в временный файл
for user in $all_users; do
    home_dir=$(getent passwd "$user" | cut -d: -f6)
    du -s "$home_dir" 2>/dev/null >> "$temp_file"
done

# Считаем средний объем домашних каталогов
avg_size=$(awk '{ total += $1 } END { print total/NR }' "$temp_file")

# Выводим заголовок таблицы
echo -e "Пользователь\tОбъем\tОтклонение (%)"

# Выводим информацию для каждого пользователя
while read -r size user_home; do
    user=$(basename "$user_home")
    deviation=$(awk -v size="$size" -v avg_size="$avg_size" 'BEGIN { print ((size - avg_size) / avg_size) * 100 }')
    printf "%s\t\t%s\t%.2f%%\n" "$user" "$size" "$deviation"
done < "$temp_file"

# Удаляем временный файл
rm "$temp_file"

====================================================================================
====================================================================================
====================================================================================

9. ﻿﻿Создать командный файл, выводящий на экран список всех процессов, запущенных заданным пользователем.
    #!/bin/bash

if [ "$#" -ne 1 ]; then
    echo "Использование: $0 <имя_пользователя>"
    exit 1
fi

username=$1

# Используем ps для вывода списка всех процессов, запущенных указанным пользователем
ps aux | grep "^$username"

====================================================================================
====================================================================================
====================================================================================

Вывести 10 самых больших файлов в файловой системы их названия и размеры

#!/bin/bash

# Используем find для поиска всех файлов в файловой системе
# Используем du для определения размера каждого файла
# Используем sort для сортировки по размеру в обратном порядке
# И выводим первые 10 строк, которые будут самыми большими файлами
find / -type f -exec du -h {} + 2>/dev/null | sort -rh | head -n 10

====================================================================================

#!/bin/bash

# Задаем высоту пирамиды
height=5

# Цикл построения пирамиды
for ((i = 1; i <= height; i++)); do
    # Выводим пробелы перед звездочками
    for ((j = 1; j <= height - i; j++)); do
        echo -n " "
    done

    # Выводим звездочки
    for ((k = 1; k <= 2 * i - 1; k++)); do
        echo -n "*"
    done

    # Переход на новую строку после завершения строки
    echo
done
==========================

#!/bin/bash

# Задаем высоту ромба (необходимо указать нечетное значение)
height=5

# Проверяем, что высота ромба нечетная
if ((height % 2 == 0)); then
    echo "Высота ромба должна быть нечетной."
    exit 1
fi

# Цикл построения верхней части ромба
for ((i = 1; i <= height / 2 + 1; i++)); do
    # Выводим пробелы перед звездочками
    for ((j = 1; j <= height / 2 + 1 - i; j++)); do
        echo -n " "
    done

    # Выводим звездочки
    for ((k = 1; k <= 2 * i - 1; k++)); do
        echo -n "*"
    done

    # Переход на новую строку после завершения строки
    echo
done

# Цикл построения нижней части ромба
for ((i = height / 2; i >= 1; i--)); do
    # Выводим пробелы перед звездочками
    for ((j = 1; j <= height / 2 + 1 - i; j++)); do
        echo -n " "
    done

    # Выводим звездочки
    for ((k = 1; k <= 2 * i - 1; k++)); do
        echo -n "*"
    done

    # Переход на новую строку после завершения строки
    echo
done
==============================
====================================================================================
====================================================================================


#!/bin/bash

# Введите ваше 4-значное число
number=1234

# Переменная для хранения флага, указывающего наличие цифры 2
has_digit_2=false

# Цикл проверки наличия цифры 2
while [ "$number" -gt 0 ]; do
    digit=$((number % 10))  # Получаем последнюю цифру
    if [ "$digit" -eq 2 ]; then
        has_digit_2=true
        break
    fi
    number=$((number / 10))  # Убираем последнюю цифру
done

# Проверяем флаг и выводим результат
if [ "$has_digit_2" = true ]; then
    echo "Число содержит цифру 2."
else
    echo "Число не содержит цифру 2."
fi
=====================
