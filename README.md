import bitcoin
import requests
import time

while True:
    # 生成一个随机的密钥
    while True:
        # 生成一个用十六进制表示的长 256 位的私钥（str类型）
        private_key = bitcoin.random_key()
        # 解码为十进制的整形密钥
        decoded_private_key = bitcoin.decode_privkey(private_key, 'hex')
        if 0 < decoded_private_key < bitcoin.N:
            break
    #print(private_key) #密钥（十六进制,长 256 位）

    # 计算地址
    # 用 WIF 格式编码密钥
    wif_encoded_private_key = bitcoin.encode_privkey(decoded_private_key, 'wif')
    # 用 01 标识的压缩密钥
    compressed_private_key = private_key + '01'
    # 生成 WIF的压缩格式
    wif_compressed_private_key = bitcoin.encode_privkey(
        bitcoin.decode_privkey(compressed_private_key, 'hex'), 'wif')
    # 计算公钥坐标 K = k * G
    public_key = bitcoin.fast_multiply(bitcoin.G, decoded_private_key)
    # 计算公钥
    hex_encoded_public_key = bitcoin.encode_pubkey(public_key, 'hex')
    # 计算压缩公钥
    # if public_key[1] % 2 == 0:  # 两种方式均可
    if public_key[1] & 1 == 0:
        compressed_prefix = '02'
    else:
        compressed_prefix = '03'
    # 转十六也可用 bitcoin.encode(xxx, 16)
    hex_compressed_public_key = compressed_prefix + hex(public_key[0])[2:]
    #print(f'压缩公钥（十六进制）{hex_compressed_public_key} '
    #      '（02 开头代表 y 是偶数，03 开头代表 y 是奇数）')
    # 传入公钥坐标对象/十六进制公钥值，输出同样的地址
    address = bitcoin.pubkey_to_address(public_key)
    #print(address) #地址（b58check）（1 开头）

    #获取钱包地址余额
    btcurl = 'https://chain.api.btc.com/v3/address/' + address # 获取余额地址
    response = requests.get(btcurl)
    BTC = (response.json()['data']['balance'])  # 科学计数法，处理返回数据

print(wif_encoded_private_key) #输出私钥
print(address) #输出地址
print(BTC) #输出地址余额
