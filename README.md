# btcjj

<code class="prism language-python has-numbering" onclick="mdcp.signin(event)" style="position: unset;"><span class="token keyword">import</span> bitcoin
<span class="token keyword">import</span> requests
<span class="token keyword">import</span> time

<span class="token keyword">while</span> <span class="token boolean">True</span><span class="token punctuation">:</span>
    <span class="token comment"># 生成一个随机的密钥</span>
    <span class="token keyword">while</span> <span class="token boolean">True</span><span class="token punctuation">:</span>
        <span class="token comment"># 生成一个用十六进制表示的长 256 位的私钥（str类型）</span>
        private_key <span class="token operator">=</span> bitcoin<span class="token punctuation">.</span>random_key<span class="token punctuation">(</span><span class="token punctuation">)</span>
        <span class="token comment"># 解码为十进制的整形密钥</span>
        decoded_private_key <span class="token operator">=</span> bitcoin<span class="token punctuation">.</span>decode_privkey<span class="token punctuation">(</span>private_key<span class="token punctuation">,</span> <span class="token string">'hex'</span><span class="token punctuation">)</span>
        <span class="token keyword">if</span> <span class="token number">0</span> <span class="token operator">&lt;</span> decoded_private_key <span class="token operator">&lt;</span> bitcoin<span class="token punctuation">.</span>N<span class="token punctuation">:</span>
            <span class="token keyword">break</span>
    <span class="token comment">#print(private_key) #密钥（十六进制,长 256 位）</span>

    <span class="token comment"># 计算地址</span>
    <span class="token comment"># 用 WIF 格式编码密钥</span>
    wif_encoded_private_key <span class="token operator">=</span> bitcoin<span class="token punctuation">.</span>encode_privkey<span class="token punctuation">(</span>decoded_private_key<span class="token punctuation">,</span> <span class="token string">'wif'</span><span class="token punctuation">)</span>
    <span class="token comment"># 用 01 标识的压缩密钥</span>
    compressed_private_key <span class="token operator">=</span> private_key <span class="token operator">+</span> <span class="token string">'01'</span>
    <span class="token comment"># 生成 WIF的压缩格式</span>
    wif_compressed_private_key <span class="token operator">=</span> bitcoin<span class="token punctuation">.</span>encode_privkey<span class="token punctuation">(</span>
        bitcoin<span class="token punctuation">.</span>decode_privkey<span class="token punctuation">(</span>compressed_private_key<span class="token punctuation">,</span> <span class="token string">'hex'</span><span class="token punctuation">)</span><span class="token punctuation">,</span> <span class="token string">'wif'</span><span class="token punctuation">)</span>
    <span class="token comment"># 计算公钥坐标 K = k * G</span>
    public_key <span class="token operator">=</span> bitcoin<span class="token punctuation">.</span>fast_multiply<span class="token punctuation">(</span>bitcoin<span class="token punctuation">.</span>G<span class="token punctuation">,</span> decoded_private_key<span class="token punctuation">)</span>
    <span class="token comment"># 计算公钥</span>
    hex_encoded_public_key <span class="token operator">=</span> bitcoin<span class="token punctuation">.</span>encode_pubkey<span class="token punctuation">(</span>public_key<span class="token punctuation">,</span> <span class="token string">'hex'</span><span class="token punctuation">)</span>
    <span class="token comment"># 计算压缩公钥</span>
    <span class="token comment"># if public_key[1] % 2 == 0:  # 两种方式均可</span>
    <span class="token keyword">if</span> public_key<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span> <span class="token operator">&amp;</span> <span class="token number">1</span> <span class="token operator">==</span> <span class="token number">0</span><span class="token punctuation">:</span>
        compressed_prefix <span class="token operator">=</span> <span class="token string">'02'</span>
    <span class="token keyword">else</span><span class="token punctuation">:</span>
        compressed_prefix <span class="token operator">=</span> <span class="token string">'03'</span>
    <span class="token comment"># 转十六也可用 bitcoin.encode(xxx, 16)</span>
    hex_compressed_public_key <span class="token operator">=</span> compressed_prefix <span class="token operator">+</span> <span class="token builtin">hex</span><span class="token punctuation">(</span>public_key<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">[</span><span class="token number">2</span><span class="token punctuation">:</span><span class="token punctuation">]</span>
    <span class="token comment">#print(f'压缩公钥（十六进制）{hex_compressed_public_key} '</span>
    <span class="token comment">#      '（02 开头代表 y 是偶数，03 开头代表 y 是奇数）')</span>
    <span class="token comment"># 传入公钥坐标对象/十六进制公钥值，输出同样的地址</span>
    address <span class="token operator">=</span> bitcoin<span class="token punctuation">.</span>pubkey_to_address<span class="token punctuation">(</span>public_key<span class="token punctuation">)</span>
    <span class="token comment">#print(address) #地址（b58check）（1 开头）</span>

    <span class="token comment">#获取钱包地址余额</span>
    btcurl <span class="token operator">=</span> <span class="token string">'https://chain.api.btc.com/v3/address/'</span> <span class="token operator">+</span> address <span class="token comment"># 获取余额地址</span>
    response <span class="token operator">=</span> requests<span class="token punctuation">.</span>get<span class="token punctuation">(</span>btcurl<span class="token punctuation">)</span>
    BTC <span class="token operator">=</span> <span class="token punctuation">(</span>response<span class="token punctuation">.</span>json<span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">[</span><span class="token string">'data'</span><span class="token punctuation">]</span><span class="token punctuation">[</span><span class="token string">'balance'</span><span class="token punctuation">]</span><span class="token punctuation">)</span>  <span class="token comment"># 科学计数法，处理返回数据</span>

<span class="token keyword">print</span><span class="token punctuation">(</span>wif_encoded_private_key<span class="token punctuation">)</span> <span class="token comment">#输出私钥</span>
<span class="token keyword">print</span><span class="token punctuation">(</span>address<span class="token punctuation">)</span> <span class="token comment">#输出地址</span>
<span class="token keyword">print</span><span class="token punctuation">(</span>BTC<span class="token punctuation">)</span> <span class="token comment">#输出地址余额</span>
<div class="hljs-button signin" data-title="登录后复制" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.4334&quot;}"></div></code>
