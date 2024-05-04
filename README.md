# Курс "Распределённые системы" Филиала МГУ в г. Севастополе (8 семестр)

## Задача 1
В транспьютерной матрице размером 4*4, в каждом узле которой находится один процесс, необходимо выполнить операцию сбора данных (длиной 4 байта) от всех процессов (`MPI_ALLGATHER`). Данные, посылаемые i-ым процессом, помещаются в i-ый элемент результирующего буфера каждого процесса. После завершения операции содержимое результирующих буферов у всех процессов должно быть одинаково.
Реализовать программу, моделирующую выполнение операции `MPI_ALLGATHER` на транспьютерной матрице при помощи пересылок `MPI` типа точка-точка.
Получить временную оценку работы алгоритма. Оценить сколько времени потребуется для выполнения операции `MPI_ALLGATHER`, если все процессы выдали ее одновременно. Время старта равно 100, время передачи байта равно 1 (`Ts=100,Tb=1`). Процессорные операции, включая чтение из памяти и запись в память, считаются бесконечно быстрыми.

### Описание решения
См. `task1/Report_task1.pdf`

### Замечания преподавателя
* В реализации присутствуют неблокирующие операции отправки `MPI_Isend`, являющиеся лишь намерением на отправку. Таким образом, необходимо использовать команды `MPI_Wait` и его аналоги.
* В программе для разных отправок используются одни и те же дескрипторы запрошенной операции связи (`status` и `request`). Так делать нельзя


## Задание 2
### Постановка задачи
Доработать MPI-программу, реализованную в рамках курса
“Суперкомпьютеры и параллельная обработка данных”. Добавить
контрольные точки для продолжения работы программы в случае сбоя.
Реализовать один из 3-х сценариев работы после сбоя:

  a) продолжить работу программы только на “исправных” процессах;

  ***b) вместо процессов, вышедших из строя, создать новые MPI-процессы,
  которые необходимо использовать для продолжения расчетов;***

  c) при запуске программы на счет сразу запустить некоторое
  дополнительное количество MPI-процессов, которые использовать в
  случае сбоя.

### Описание решения
Из доступных на старте программы процессов будем оставлять некоторое
количество в качестве запасных. Когда какой-нибудь из работающих
процессов упадет, будем выделять один из запасных процессов на его место,
а итерацию перезапускать на новом коммуникаторе, полученном из старого
путем удаления упавших процессов. Пересоздание коммуникатора в таком
случае можно запускать после каждой итерации.

Чтобы падения процессов не приводили к падению программы,
зарегистрируем обработчик на возврат кодов ошибок из MPI_ функций
взаимодействия. На возвращаемые значения функций можно 
не смотреть, т.к. на каждой итерации 
у нас происходит `MPI_Comm_shrink()`, после которого
все процессы в коммуникаторе будут живыми.

Операция `MPI_Comm_shrink()` может перенумеровать процессы,
поэтому после её вызова нужно обновлять ранки.

Чтобы сохранять промежуточные вычисления, после каждой итерации для
всех работающих процессов будем сохранять матрицы `A` и `B` в файлы. В
начале итерации будем читать файлы, соответствующие процессу с
указанным ранком.

### Замечания преподавателя
* Неправильно прописанный обработчик ошибок
* Процессы узнают о сбое слишком поздно