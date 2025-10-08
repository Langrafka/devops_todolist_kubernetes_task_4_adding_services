## Інструкція з валідації ToDo App у Kubernetes (Task 3)

Ця інструкція показує, як розгорнути та протестувати додаток, використовуючи створені маніфести.

### 1. Збірка та Публікація Образу (Критично)

Обов'язково виконайте ці команди. **ЗАМІНІТЬ** плейсхолдер **`{your-dockerhub-username}`** на ваш фактичний логін Docker Hub у командах нижче, а також у файлі `todoapp-pods.yml`.

1.  **Зібрати образ:**
    ```bash
    docker build . -t {your-dockerhub-username}/todoapp:3.0.0
    ```
2.  **Завантажити образ на Docker Hub:**
    ```bash
    docker push {your-dockerhub-username}/todoapp:3.0.0
    ```

---

### 2. Застосування маніфестів

Усі маніфести знаходяться у вашій робочій директорії. Застосовуйте їх по черзі:

1.  **Створити простір імен `todoapp`:**
    ```bash
    kubectl apply -f namespace.yml
    ```
2.  **Застосувати Pod-и (todoapp-1, todoapp-2) та Busybox:**
    ```bash
    kubectl apply -f todoapp-pods.yml -n todoapp
    kubectl apply -f busybox.yml -n todoapp
    ```
3.  **Створити сервіс ClusterIP:**
    ```bash
    kubectl apply -f clusterip.yml -n todoapp
    ```
4.  **Створити сервіс NodePort:**
    ```bash
    kubectl apply -f nodeport.yml -n todoapp
    ```
5.  **Перевірити статус Pod-ів та Сервісів:**
    ```bash
    kubectl get pods -n todoapp
    kubectl get svc -n todoapp
    ```

---

### 3. Тестування (Внутрішні та Зовнішні Запити)

#### A. Тестування ClusterIP Service через `port-forward` (Task 9)

1.  **Прокинути порт 8080 локально (залишити термінал відкритим):**
    ```bash
    kubectl port-forward service/todoapp-clusterip-svc 8080:80 -n todoapp
    ```
2.  **Перевірити доступ (У новому вікні терміналу):**
    ```bash
    curl http://localhost:8080/
    ```

#### B. Тестування ClusterIP DNS всередині кластера (Task 8)

**Важливо:** Оскільки образ BusyBox не містить `curl`, ми використовуємо `wget`.

1.  **Зайти всередину контейнера busybox:**
    ```bash
    kubectl exec -it busybox -n todoapp -- /bin/sh
    ```
2.  **Всередині busybox (виконати wget до DNS-імені ClusterIP):**
    ```bash
    wget -qO- [http://todoapp-clusterip-svc.todoapp.svc.cluster.local/](http://todoapp-clusterip-svc.todoapp.svc.cluster.local/)
    ```
3.  **Вийти з busybox:**
    ```bash
    exit
    ```

#### C. Тестування через NodePort Service (Task 10)

1.  **Перевірити виділений порт:**
    ```bash
    kubectl get svc todoapp-nodeport-svc -n todoapp
    # Знайдіть NodePort (наприклад, 30007)
    ```
2.  **Звернутися до додатка ззовні кластера:**
    ```bash
    # Для локальних кластерів (Docker Desktop, Minikube):
    curl http://localhost:30007/
    ```