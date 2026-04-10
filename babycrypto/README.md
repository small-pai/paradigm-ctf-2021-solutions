# Babycrypto

## 漏洞类型
ECDSA k 值重用（Nonce Reuse Attack）

## 攻击原理
`chal.py` 中使用固定的 `session_secret` 作为 ECDSA 签名的 k 值，导致可以从两个不同消息的签名中恢复私钥。

## 攻击步骤

1. 获取签名预言机返回的 4 组签名数据
2. 使用前两个签名 (AAAA, BBBB) 恢复 k 值和私钥
3. 计算挑战消息的签名
4. 提交签名获取 flag

## 关键代码

```python
# 恢复 k
k = ((z1 - z2) % n) * inverse_mod((s1 - s2) % n, n) % n

# 恢复私钥
d = (((s1 * k) % n - z1) % n) * inverse_mod(r1, n) % n

# 计算挑战签名
r = (k * G).x() % n
s = inverse_mod(k, n) * ((z3 + d * r) % n) % n
```
## Flag
PCTF{placeholder}
