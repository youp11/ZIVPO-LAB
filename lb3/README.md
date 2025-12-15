```
using System;
using System.IO;
using System.Runtime.InteropServices;

namespace ZIVPOSlab
{
    class Program
    {
        // Системный вызов
        [DllImport("kernel32.dll", SetLastError = true)]
        static extern IntPtr CreateFile(
            string lpFileName,
            uint dwDesiredAccess,
            uint dwShareMode,
            IntPtr lpSecurityAttributes,
            uint dwCreationDisposition,
            uint dwFlagsAndAttributes,
            IntPtr hTemplateFile);

        static void Main()
        {
            // Консоль

            Console.WriteLine("Лабораторная работа №3");
            Console.WriteLine("Выполнили: Shishkov Vaukin Matushkin");

            
            // Системный вызов - создаем файл test.txt
            string testFile = Path.Combine(
                Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments), 
                "test_lab3.txt");
            
            IntPtr fileHandle = CreateFile(testFile, 0x80000000, 0,
                IntPtr.Zero, 2, 0x80, IntPtr.Zero);
            
            Console.WriteLine(fileHandle != (IntPtr)(-1) ?
                "✓ Системный вызов CreateFile выполнен успешно" :
                "✗ Ошибка системного вызова CreateFile");
            
            // Вывод файлов средствами .NET
            string docsPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
            Console.WriteLine("\nСодержимое папки Documents:");
            Console.WriteLine("Путь: " + docsPath);
            
            Console.WriteLine("\n--- Папки ---");
            try
            {
                foreach (var dir in Directory.GetDirectories(docsPath))
                    Console.WriteLine("  [DIR]  " + Path.GetFileName(dir));
            }
            catch { }
            
            Console.WriteLine("\n--- Файлы ---");
            try
            {
                foreach (var file in Directory.GetFiles(docsPath))
                    Console.WriteLine("  [FILE] " + Path.GetFileName(file));
            }
            catch { }
            Console.WriteLine("\nНажмите любую клавишу для выхода...");
            Console.ReadKey();
        }
    }
}
```
<img src="https://github.com/youp11/ZIVPO-LAB/blob/main/lb3/image/76f04249-0541-4b9c-b345-c8da18c6ef93.png" width="800" height="600">
<img src="https://github.com/youp11/ZIVPO-LAB/blob/main/lb3/image/01ebcc98-a3e4-42c3-9c9c-229a7db88ad9.png" width="800" height="600">

## Описание работы программы

Программа открывает консоль и выводит информацию о лабораторной работе, а также о студентах, её выполнивших.
Далее с использованием **Windows API** (функция `CreateFile` из библиотеки `kernel32.dll`) создаётся временный файл **`test.txt`** в папке **Documents** текущего пользователя.
После этого программа:
- определяет путь к папке *Documents* текущего пользователя;
- считывает все файлы и подкаталоги, находящиеся в данной директории;
- выводит полученный перечень в консоль.
Для предотвращения автоматического закрытия консольного окна после завершения выполнения всех операций программа ожидает **нажатия любой клавиши пользователем**.
<img src="https://github.com/youp11/ZIVPO-LAB/blob/main/lb3/image/5429b06b-3d8a-4f49-949c-c792c2c634ee.png" width="600" height="800">
## Анализ с использованием IDA
На заключительном этапе исходный код компилируется в исполняемый файл формата **`.exe`**.  
При анализе полученного файла с помощью **IDA** видно, что **фамилии остаются в читаемом виде**, что указывает на отсутствие обфускации строк в бинарном файле.
<img src="https://github.com/youp11/ZIVPO-LAB/blob/main/lb3/image/067ea49e-4ff1-4623-87b0-8d7505440102.png" width="800" height="600">
## Обфускация программы
Далее используем программу‑обфускатор, в нашем случае **yetAnotherObfuscator**.
<img src="https://github.com/youp11/ZIVPO-LAB/blob/main/lb3/image/a2e57dad-bb7c-47d7-af22-06c299b74956.png" width="800" height="600">
После сравнения двух дизассемблированных файлов очевидны ключевые изменения, внесённые обфускацией.
<img src="https://github.com/youp11/ZIVPO-LAB/blob/main/lb3/image/d0788cf6-dc9d-4fdf-808b-5d2ef1ba1d17.png" width="800" height="600">
В оригинальном коде все строки — заголовок лабораторной, имена студентов, сообщения об ошибках, имя файла `test.txt`, текстовые разделители — хранились в открытом виде.
После обфускации эти строковые литералы были полностью удалены и заменены на наборы данных, напоминающие строки в кодировке **Base64**. Для работы с ними в код была добавлена специальная функция‑дешифратор. Теперь при каждом обращении к строке программа сначала вызывает эту функцию, передаёт ей зашифрованную константу и получает обратно исходный текст.
Это изменение непосредственно отразилось на логике программы: вызовы методов вроде `Console.WriteLine`, `Path.Combine` или `String.Concat` теперь используют обёртку — вместо прямого строкового литерала они получают аргумент, который является результатом работы функции дешифровки.
Таким образом, обфускация скрыла весь текстовый контент программы, сделав его недоступным при статическом анализе и восстанавливаемым только в процессе выполнения.
