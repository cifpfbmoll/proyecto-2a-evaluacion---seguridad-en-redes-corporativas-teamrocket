Obligación a elevar privilegios una vez iniciada sesión

```PermitRootLogin no```

Establecer un máximo de intentos de password

```MaxAuthTries 3```

Desactivar autenticación por host, obligando a usar contraseña

```bash
HostbasedAutentication no
IgnoreRhosts yes
```

Prohibir passwords vacíos

```PermitEmptyPasswords no```

PAM

``` UsePAM yes```

Definir máximo de sesiones a la vez

```MaxStartups 10:30:60```




