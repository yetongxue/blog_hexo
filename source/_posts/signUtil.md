---
title: 签名用的工具箱
date: 2017-01-07 22:55:44
updated: 
categories: code
permalink:
tags: [签名,工具类]
---
签名算法工具，签名生成的步骤如下：
 第一步，设所有发送或者接收到的数据为集合M，将集合M内非空参数值的参数按照参数名ASCII码从小到大排序（字典序），使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串stringA。

 第二步，在stringA最后拼接上key得到stringSignTemp字符串，并对stringSignTemp进行MD5运算，再将得到的字符串所有字符转换为大写，得到sign值signValue。

特别注意以下重要规则：</br>
	◆ 参数名ASCII码从小到大排序（字典序）；</br>
	◆ 如果参数的值为空不参与签名；</br>
	◆ 参数名区分大小写；</br>
	◆ 验证调用返回或主动通知签名时，传送的sign参数不参与签名，将生成的签名与该sign值作校验。</br>
	◆ key值由互金端提供；</br>

```
	import java.io.StringReader;
	import java.net.URLEncoder;
	import java.util.ArrayList;
	import java.util.Collections;
	import java.util.HashMap;
	import java.util.List;
	import java.util.Map;
	import java.util.SortedMap;
	import java.util.TreeMap;
	
	import org.dom4j.Document;
	import org.dom4j.DocumentException;
	import org.dom4j.Element;
	import org.dom4j.io.SAXReader;
	import org.xml.sax.InputSource;
	/**
	* ClassName:SignUtils</br>
	* Function: 签名用的工具箱</br>
	* Date:     2017-2-27 下午3:22:33 </br>
	* <功能详细描述>签名算法</br>
	* @author    
	*/

	public class SignUtils {
	    public static void main(String[] args) {
	        String key = "CvRu8Dydbe7SETJoqh5wQYBp4PnZxigz";
	        SortedMap<String,String> params = new TreeMap<String,String>();
	        params.put("orderId", "1112233");
	        params.put("merchantId", "3700603014");
	        params.put("applyAmount", "1000000");
	        params.put("sign", "sdfsdf");
	        String sign = getSign(params,key);//签名
	        System.out.println("sign:"+sign.toUpperCase());
	        params.put("sign", sign.toUpperCase());
	        System.out.println("sign:"+checkParam(params,key));//验证
	    }
	    /** <一句话功能简述>
	     * <功能详细描述>验证返回参数
	     * @param params
	     * @param key
	     * @return
	     * @see [类、类#方法、类#成员]
	     */
	    public static boolean checkParam(Map<String,String> params,String key){
	        boolean result = false;
	        if(params.containsKey("sign")){
	            String sign = params.get("sign");
	            params.remove("sign");
	            params = paraFilter(params);
	            StringBuilder buf = new StringBuilder((params.size() +1) * 10);
	            SignUtils.buildPayParams(buf,params,false);
	            String preStr = buf.toString();
	            String signRecieve = MD5.sign(preStr, "&key=" + key, "utf-8");
	            System.out.println(signRecieve);
	            result = sign.equalsIgnoreCase(signRecieve);
	        }
	        return result;
	    }
	    
	    /**
	     * 获取签名
	     * @param map 对象
	     * @param key 密匙
	     * @return 签名
	     */
	    public static String getSign(SortedMap<String,String> map,String key){
	        Map<String, String> params = SignUtils.paraFilter(map);
	        StringBuilder buf = new StringBuilder((params.size() + 1) * 10);
	        SignUtils.buildPayParams(buf, params, false);
	        String preStr = buf.toString();
	        System.out.println(preStr);
	        String sign = MD5.sign(preStr, "&key=" + key, "utf-8").toUpperCase();
	        return sign;
	    }
	    
	    /**
	     * 过滤参数:过滤掉sArray中value为空和key为sign的键值对
	     * @param sArray
	     * @return
	     */
	
	    public static Map<String, String> paraFilter(Map<String, String> sArray) {
	        Map<String, String> result = new HashMap<String, String>(sArray.size());
	        if (sArray == null || sArray.size() <= 0) {
	            return result;
	        }
	        for (String key : sArray.keySet()) {
	            String value = sArray.get(key);
	            if (value == null || value.equals("") || key.equalsIgnoreCase("sign")) {
	                continue;
	            }
	            result.put(key, value);
	        }
	        return result;
	    }
	    
	
	   /**
	    * 将集合M内非空参数值的参数按照参数名ASCII码从小到大排序（字典序），使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串stringA
	    * @param sb 返回的字符串
	    * @param payParams  集合M
	    * @param encoding 是否需要经过UTF-8编码：true-需要  false-不需要
	    */
	
	    public static void buildPayParams(StringBuilder sb,Map<String, String> payParams,boolean encoding){
	        List<String> keys = new ArrayList<String>(payParams.keySet());
	        Collections.sort(keys);
	        for(String key : keys){
	            sb.append(key).append("=");
	            if(encoding){
	                sb.append(urlEncode(payParams.get(key)));
	            }else{
	                sb.append(payParams.get(key));
	            }
	            sb.append("&");
	        }
	        sb.setLength(sb.length() - 1);
	    }
	    
	    public static String urlEncode(String str){
	        try {
	            return URLEncoder.encode(str, "UTF-8");
	        } catch (Throwable e) {
	            return str;
	        }
	    }
	    
	    
	    public static Element readerXml(String body,String encode) throws DocumentException {
	        SAXReader reader = new SAXReader(false);
	        InputSource source = new InputSource(new StringReader(body));
	        source.setEncoding(encode);
	        Document doc = reader.read(source);
	        Element element = doc.getRootElement();
	        return element;
	    }
	
	}
```
