#SRC #Sem1 #TIWPR


| URI                                  | GET                    | POST                  | PUT                     | PATCH                    | DELETE                |
| ------------------------------------ | ---------------------- | --------------------- | ----------------------- | ------------------------ | --------------------- |
| /api/auth/tokens                     | x                      | logowanie użytkownika | x                       | x                        | x                     |
| /api/users                           | x                      | tworzenie konta       | x                       | x                        | x                     |
| /api/user/settings                   | ustawienia użytkownika | x                     | x                       | aktualizacja ustawień    | x                     |
| /api/files                           | lista plików           | dodanie pliku         | x                       | x                        | x                     |
| /api/files/{id}                      | metadane o pliku       | x                     | aktualizacja metadanych | x                        | usunięcie pliku       |
| /api/files/{id}/raw                  | pobieranie pliku       | x                     | aktualizacja pliku      | x                        | x                     |
| /api/task-list                       | lista list zadań       | dodanie listy zadań   | x                       | x                        | x                     |
| /api/tasks-list/{id}                 | pewna lista zadań      | dodanie listy zadań   | x                       | aktualizacja listy zadań | usunięcie listy zadań |
| /api/tasks-list/{list_id}/tasks      | zadania z danej listy  | dodanie zadania       | x                       | x                        | x                     |
| /api/tasks-list/{list_id}/tasks/{id} | pojedyńcze zadanie     | x                     | x                       | aktualizacja zadania     | usunięcie zadania     |
| /api/tasks-lists-merges              | x                      | zcalenie dwóch list   | x                       | x                        | x                     |
