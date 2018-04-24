### java rsa 签名实现 ###

### 1、基于openssl生成RSA公私钥对

 * 工具下载：从网上下载openssl工具:http://www.slproweb.com/products/Win32OpenSSL.html

 * 生成私钥：进入到openssl的bin目录下，执行以下命令

 ```
 openssl genrsa -out rsa_private_key.pem 1024
 ```

* 生成公钥:在bin目录下，执行以下命令

 ```
 openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
 ```

### java 签名方法实现

 注意：参数privateKey是Pem私钥文件中去除头（-----BEGIN RSA PRIVATE KEY-----）和尾（-----END RSA PRIVATE KEY-----）以及换行符后的字符串。

```java
/***
 * RSA签名工具类
 *
 * @author majun@12301.cc
 *
 * @date 2018年1月15日
 */
public class RsaSignatureUtils {

	private static final String RSA_ALGORITHM = "RSA";

	private static final String ALGORITHM = "SHA256WITHRSA";

	private static final String CHARSET = "UTF-8";

	// 公钥
	private static final String PUBLICKEY ="";

	private static final String privatekey = "";

	/***
	 * 字符串RSA算法签名
	 *
	 * @param content
	 *            需要签名的字符串
	 * @param privateKey
	 *            私钥
	 * @return RSA签名后的字符串
	 * @throws SignatureException
	 */
	public static String rsaSign(String content, String privateKey) throws SignatureException {

		try {
			PrivateKey priKey = getPrivateKeyFromPKCS8(RSA_ALGORITHM, new ByteArrayInputStream(privateKey.getBytes(CHARSET)));

			Signature signature = Signature.getInstance(ALGORITHM);

			signature.initSign(priKey);

			signature.update(content.getBytes(CHARSET));

			byte[] signed = signature.sign();

			return new String(Base64.encodeBase64(signed));

		} catch (Exception e) {
			throw new SignatureException("RSAcontent = " + content + "; charset = " + CHARSET, e);
		}
	}

	/***
	 * rsa签名验证
	 *
	 * @param content
	 *            被签名的内容
	 * @param sign
	 *            签名结果
	 * @param publicKey
	 *            rsa公钥
	 * @return
	 * @throws SignatureException
	 */
	public static boolean rsaCheck(String content, String sign, String publicKey) throws SignatureException {

		try {
			PublicKey pubKey = getPublicKeyFromX509(RSA_ALGORITHM, new ByteArrayInputStream(publicKey.getBytes()));

			Signature signature = Signature.getInstance(ALGORITHM);

			signature.initVerify(pubKey);

			signature.update(getContentBytes(content, CHARSET));

			return signature.verify(Base64.decodeBase64(sign.getBytes()));
		} catch (Exception e) {
			throw new SignatureException("RSA验证签名[content = " + content + "; charset = " + CHARSET + "; signature = " + sign + "]发生异常!", e);
		}

	}

	public static boolean rsaCheck(String content, String sign) throws SignatureException {
		return rsaCheck(content, sign, PUBLICKEY);
	}

	public static <T extends Serializable> boolean rsaCheck(T t, String sign) throws SignatureException {
		String content = JSON.toJSONString(t);
		return rsaCheck(content, sign, PUBLICKEY);
	}



	/***
	 * 获取私钥key
	 *
	 * @param algorithm
	 *            签名算法
	 * @param ins
	 *            输入流
	 * @return
	 * @throws Exception
	 */
	private static PrivateKey getPrivateKeyFromPKCS8(String algorithm, InputStream ins) throws Exception {

		if (ins == null || StringUtils.isEmpty(algorithm)) {
			return null;
		}

		KeyFactory keyFactory = KeyFactory.getInstance(RSA_ALGORITHM);

		byte[] encodedKey = inputStrem2byte(ins);

		encodedKey = Base64.decodeBase64(encodedKey);

		return keyFactory.generatePrivate(new PKCS8EncodedKeySpec(encodedKey));
	}

	/****
	 * 获取公钥key
	 *
	 * @param algorithm
	 *            签名算法
	 * @param ins
	 *            输入流
	 * @return
	 * @throws Exception
	 */
	private static PublicKey getPublicKeyFromX509(String algorithm, InputStream ins) throws Exception {

		KeyFactory keyFactory = KeyFactory.getInstance(algorithm);

		byte[] encodedKey = inputStrem2byte(ins);

		encodedKey = Base64.decodeBase64(encodedKey);

		return keyFactory.generatePublic(new X509EncodedKeySpec(encodedKey));
	}

	/***
	 * 将InputStrem转换成byte数组
	 *
	 * @param inStream
	 * @return
	 * @throws IOException
	 */
	private static byte[] inputStrem2byte(InputStream inStream) throws IOException {

		ByteArrayOutputStream swapStream = new ByteArrayOutputStream();

		byte[] buff = new byte[1024];

		int rc = 0;

		while ((rc = inStream.read(buff, 0, 1024)) > 0) {
			swapStream.write(buff, 0, rc);
		}

		byte[] in2b = swapStream.toByteArray();

		return in2b;
	}

	/****
	 * 将字符串转换成byte数组
	 *
	 * @param content
	 *            字符串
	 * @param charset
	 *            编码
	 * @return
	 * @throws UnsupportedEncodingException
	 */
	private static byte[] getContentBytes(String content, String charset) throws UnsupportedEncodingException {
		if (StringUtils.isEmpty(charset)) {
			return content.getBytes();
		}
		return content.getBytes(charset);
	}
}
```

如果签名报以下错误：

java.security.spec.InvalidKeySpecException: java.security.InvalidKeyException: IOException : algid parse error, not a sequence

则说明rsa私钥的格式不是pksc8格式，需要使用以下命令转换一下：

openssl pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM -nocrypt

然后再提取去除头和尾以及换行符后字符串作为java版用的rsa私钥。
