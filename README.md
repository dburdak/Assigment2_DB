# Assigment2_DB
## Кроки оптимізації
Запит підраховує к-ть студентів, що мають вищу оцінку за середню для статі та їх соціально-економічного статусу.
<img width="612" alt="Знімок екрана 2025-06-05 о 17 34 06" src="https://github.com/user-attachments/assets/7235a4c3-4d4f-4b58-80a6-7dfd07ef9e02" />

### Початковий запит:
  <img width="392" alt="Знімок екрана 2025-06-05 о 17 25 20" src="https://github.com/user-attachments/assets/8696e59e-2508-4814-9aa7-35fceeda9fb8" />
  
### Оптимізований:
  <img width="567" alt="Знімок екрана 2025-06-05 о 18 42 39" src="https://github.com/user-attachments/assets/5d469810-bb15-4c5f-a63d-622dc9042971" />

1.Першочергово можемо побачити, що повторюється фільтр, отже його треба винести як CTE
  <img width="832" alt="Знімок екрана 2025-06-05 о 17 26 01" src="https://github.com/user-attachments/assets/bafa6cb6-d32a-4577-9447-ed0b8e6c7525" />

2. В запиті також присутній нелогічний JOIN
   
   <img width="591" alt="Знімок екрана 2025-06-05 о 17 59 13" src="https://github.com/user-attachments/assets/2d061043-cc3d-4e6c-b8a8-09fe16da6e5d" />

   Замінила його на підзапит:
  <img width="586" alt="Знімок екрана 2025-06-05 о 18 00 02" src="https://github.com/user-attachments/assets/4ff07980-8e9f-466d-89ee-a66d6c9f8c27" />


4. Далі, так як, логіка присутня і немає зайвих підзапитів чи частин коду, встановлюємо індекс
  <img width="621" alt="Знімок екрана 2025-06-05 о 18 00 38" src="https://github.com/user-attachments/assets/7f63cbb4-c771-408b-b2ba-97f7bbac64d0" />
  
  Я вирішила створити саме covered index, адже запит обирає невелику к-ть стовпців і якщо ми його створимо це тільки пришвидшить виконання, а не сповільнить, за рахунок принципу         роботи covered index.

## Порівняння EXECUTION PLAN
Спочатку
``` -> -> Table scan on <temporary>  (actual time=3539..3539 rows=6 loops=1)
    -> Aggregate using temporary table  (actual time=3539..3539 rows=6 loops=1)
        -> Filter: ((f.Age >= 16) and (f.SES_Quartile >= 2) and (f.TestScore_Math > '74.98159968928803'))  (cost=807018 rows=293352) (actual time=1495..2955 rows=1.8e+6 loops=1)
            -> Covering index scan on f using idx_filtering  (cost=807018 rows=7.92e+6) (actual time=0.036..2626 rows=8e+6 loops=1)

```
загальний час виконання - 3539 ms

Після покращень:
```
-> Table scan on <temporary>  (actual time=2178..2178 rows=6 loops=1)
    -> Aggregate using temporary table  (actual time=2178..2178 rows=6 loops=1)
        -> Filter: ((train.TestScore_Math > (select #3)) and (train.Age >= 16) and (train.SES_Quartile >= 2))  (cost=814302 rows=440073) (actual time=27..1487 rows=1.8e+6 loops=1)
            -> Covering index range scan on train using covering over (2 <= SES_Quartile AND 16 <= Age)  (cost=814302 rows=3.96e+6) (actual time=0.0234..1209 rows=5.2e+6 loops=1)
            -> Select #3 (subquery in condition; run only once)
                -> Aggregate: avg(train.TestScore_Math)  (cost=1.21e+6 rows=1) (actual time=1710..1710 rows=1 loops=1)
                    -> Filter: ((train.Age >= 16) and (train.SES_Quartile >= 2))  (cost=902330 rows=1.32e+6) (actual time=0.0845..1622 rows=3.6e+6 loops=1)
                        -> Covering index range scan on train using covering over (2 <= SES_Quartile AND 16 <= Age)  (cost=902330 rows=3.96e+6) (actual time=0.0818..1397 rows=5.2e+6 loops=1)
```
загальний час виконання - 2178 ms
