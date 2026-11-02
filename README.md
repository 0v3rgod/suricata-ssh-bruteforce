# Suricata SSH Brute-Force Rules

Regra custom do Suricata para detectar ataques de força bruta SSH. Testada em laboratório VirtualBox (máquina Linux atacando VM Windows 11 com OpenSSH Server).

## Regras incluídas

| SID | Descrição | Trigger |
|-----|-----------|---------|
| 1000001 | Muitas conexões SSH novas | 10 conexões/60s do mesmo IP |
| 1000002 | Falhas de senha | 5 "Failed password"/30s |
| 1000003 | Usuários inválidos | 5 "Invalid user"/30s |
| 1000004 | Scanner (conexão sem auth) | 1 conexão SSH que não autentica |

## Instalação

```bash
# Copie para o diretório de regras do Suricata
sudo cp ssh-bruteforce.rules /etc/suricata/rules/

# Atualize e recarregue
sudo suricata-update
sudo systemctl restart suricata
```

## Verificação

```bash
# Teste: rode hydra do attacker contra o target
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.101

# No Suricata, veja os alerts
sudo tail -f /var/log/suricata/eve.json | jq 'select(.alert.signature_id >= 1000001 and .alert.signature_id <= 1000004)'
```

## Lab de teste

- **Attacker**: Kali Linux (VirtualBox, NAT network)
- **Target**: Windows 11 + OpenSSH Server (MSI v10.0.0.0, porta 22 só no tailnet 100.64.0.0/10)
- **IDS**: Suricata 7.x no attacker (interface `eth0` em modo promíscuo no VirtualBox)

## Notas

- `detection_filter` com `track by_src` evita falsos positivos de conexões legítimas
- `classtype:attempted-admin` permite correlacionar com outras tentativas de admin
- Regras não usam `threshold` (obsoleto) — só `detection_filter`

## Licença

MIT — uso educacional/laboratório.
