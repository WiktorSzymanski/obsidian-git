#SRC #Sem1 #TIWPR


| URI                                  | GET                    | POST                  | PUT | PATCH                    | DELETE                |
| ------------------------------------ | ---------------------- | --------------------- | --- | ------------------------ | --------------------- |
| /api/auth/login                      | x                      | logowanie użytkownika | x   | x                        | x                     |
| /api/auth/register                   | x                      | tworzenie konta       | x   | x                        | x                     |
| /api/files                           | lista plików           | dodanie pliku         | x   | x                        | x                     |
| /api/files/{id}                      | pobieranie pliku       | x                     | x   | x                        | usunięcie pliku       |
| /api/task-list                       | lista list zadań       | dodanie listy zadań   | x   | x                        | x                     |
| /api/tasks-list/{id}                 | pewna lista zadań      | dodanie listy zadań   | x   | aktualizacja listy zadań | usunięcie listy zadań |
| /api/tasks-list/{list_id}/tasks      | zadania z danej listy  | dodanie zadania       | x   | x                        | x                     |
| /api/tasks-list/{list_id}/tasks/{id} | pojedyńcze zadanie     | x                     |     | aktualizacja zadania     | usunięcie zadania     |
| /api/settings                        | ustawienia użytkownika | x                     | x   | aktualizacja ustawień    | x                     |
