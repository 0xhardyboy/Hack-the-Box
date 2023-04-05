# Hack the Box - Traverxec

```CSS
Machine IP: 10.10.10.165 - Linux
Webserver: Nostromo 1.9.6

Vulnerabilities:
  - CVE-2019-16278 (Nostromo: Path Traversal and Command Execution)
```

## Server:
- Nostromo 1.9.6
![image](https://user-images.githubusercontent.com/83878909/229997880-3f1414ac-da7d-48ab-8025-146541085139.png)

## Port-80 (HTTP)
- Home Page
![image](https://user-images.githubusercontent.com/83878909/229997572-6322b2e8-9af1-4c06-aad8-c706c98a0b27.png)

---

# Exploit
![image](https://user-images.githubusercontent.com/83878909/230005962-a408db0c-096d-4409-87fd-54a513a69896.png)

### Path Traversal and Command Execution
![image](https://user-images.githubusercontent.com/83878909/230006489-a47d88d4-c2f0-40ea-baaa-5ee8930fe14c.png)

![image](https://user-images.githubusercontent.com/83878909/230008238-17e20208-9f10-48a8-b2c6-aa782ddc1645.png)

### Shell Upgrade
![image](https://user-images.githubusercontent.com/83878909/230012750-9484c874-7daf-4e16-a42e-24f309abed30.png)
