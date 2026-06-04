# 3. Налаштування брандмауера

Ubuntu має UFW.

Дозволити:

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

Увімкнути:

```bash
sudo ufw enable
```

Перевірити:

```bash
sudo ufw status
```

---

# 4. Встановлення Fail2Ban

Захист від перебору паролів.

```bash
sudo apt install fail2ban -y
```

Запуск:

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Перевірка:

```bash
sudo fail2ban-client status
```

---



# 8. Захист панелі CloudPanel

Порт 8443 видно всьому інтернету.

Краще обмежити доступ.

Наприклад дозволити тільки свою IP:

```bash
sudo ufw allow from ВАША_IP to any port 8443
sudo ufw deny 8443
```

Тоді панель буде доступна лише тобі.

---