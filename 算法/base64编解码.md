# base64编解码介绍

## 使用
&emsp;&emsp;最近在工作中遇到了需要在`url`中传递参数的问题, 所以用到了`base64`编码.
&emsp;&emsp;Base64是网络上最常见的用于传输8Bit字节码的编码方式之一，Base64就是一种基于64个可打印字符来表示二进制数据的方法。 Base64并不是压缩算法, 也不是HASH算法, 而是根据一定可以进行无损的编码解码用于数据传递.

## 规则
* 1 基于64个可打印字符, 通常也就是`[a-Z1-9]+/`共计64个
* 2 Base64要求把每三个8Bit的字节转换为四个6Bit的字节, 所以需要64个可打印字符; 并且编码后会变得更长
* 3 每76个字符加一个换行符(RFC822)
* 4 如果原本的字符数不是3的倍数 那么原文不足的位数补0; 在末尾增加`=`号确保编码后长度是4的倍数

## 注意
* 1 以上仅仅是标准的说法, 实际上由于我们使用用的不同, `+/=`号都可以不同; 甚至我们可以不进行补全到4的倍数

## 原理
* 1 首先确定64个字符, 如果我们要用于URL传递, 那么`+/=`都不可用了; 我们可以使用`[a-Z0-9-_]`并且末尾不添加等号
* 2 编码时, 每次取出三个字节放入一个24位的缓冲区中, 每次取6位高两位补0(其值必然小于64), 然后对应为一个字符
* 3 解码时, 每次取出4个字节进行位操作

## 编码实现
*由于笔者需要用lua实现, 所以这里使用C实现一个标准的, 然后使用lua实现一个可以用于HTTP的; 代码上基本相差不大*
```
#include <stdlib.h>

const char *base64_list = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ123456789+/";
const char base64_last 	= '=';

char* base64_enc(const char* data, int len, char *dst)
{
	if(!data || len <= 0)	return NULL;

	int new_len = (len * 4) / 3 + 5;
	if(!dst)
		dst = (char*)calloc(new_len, 1);

	int index = 0;

	int i = 0;
	for(i = 0; i < len; i+=3)
	{
		char tmp[3] = { 0 };
		for(int j = 0; j < 3 && j < (len-i); j++)
		{
			tmp[j] = data[i+j]; 
		}
		dst[index++] = base64_list[tmp[0]>>2];
		dst[index++] = base64_list[((tmp[0]&3)<<4) + (tmp[1]>>4)];
		dst[index++] = base64_list[((tmp[1]&0xf)<<2) + (tmp[2]>>6)];
		dst[index++] = base64_list[tmp[2]&0x3f];
	}
	//注意这里是允许data不以\0结尾的

	if(i == len + 1)
		dst[index - 1] = '=';
	else if(i == len + 2)
		dst[index - 1] = dst[index - 2] = '=';

	return dst;	
}

char* base64_dec(const char *data, int len, char *dst)
{
	if(!data || len < 0)
		return NULL;
	int new_len = (len * 3) / 4 + 3;
	if(!dst)
		dst = (char*)calloc(new_len, 1);

	u_int8_t map[128] = { 0 };
	for(int i = 0; i < 64; i++)
	{
		map[base64_list[i]] = i;
	}

	if(data[len-1] == '=') len--;
	if(data[len-2] == '=') len--;

	int index = 0;
	for(int i = 0; i < len; i+=4)
	{
		u_int8_t tmp[4] = { 0 };
		for(int j = 0; j < 4 && j < (len - i); j++)
		{
				tmp[j] = map[data[i+j]];
		}

		dst[index++] = (tmp[0]<<2) + (tmp[1]>>4);
		if(len - i > 2)
			dst[index++] = (tmp[1]<<4) + (tmp[2]>>2);
		if(len - i > 3)
			dst[index++] = (tmp[2]<<6) + tmp[3];

	}

	return dst;
}

#ifdef TEST_BASE64
#include <string.h>
#include <stdio.h>
	int main()
	{
		char buffer[128];
		scanf("%s", &buffer);

		char *enc = base64_enc(buffer, strlen(buffer), NULL);
		printf("base encode: %s\n", enc);;

		char *dec = base64_dec(enc, strlen(enc), NULL);
		printf("base decode: %s\n", dec);

		return 1;
	}

#endif
```

## 额外lua的实现, 可用于URL传递
```
local b = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_/'

-- encoding
function enc(data)
    return ((data:gsub('.', function(x) 
        local r, a = '', x:byte()
        for i = 8, 1, -1 do
            r = r .. (a % 2^i - a % 2^(i-1) > 0 and '1' or '0')
        end
        --1, 高字节在前
        --2,  a%2^4=a%00010000=0000xxxx相当于和00001111进行与操作, (a % 2^i - a % 2^(i-1) > 0就可以判断第i位是否是1

        return r
    end) .. '0000'):gsub('%d%d%d?%d?%d?%d?', function(x)
        --每三个字节转变为4个字节, 如果多出1个8位扩展为12位, 如果多出2个16位扩展为18位; 所以最多添加4个0
        if (#x < 6) then 
            return '' 
        end

        local index = 0
        for i = 1, 6 do 
            index = index + (x:sub(i, i) == '1' and 2^(6-i) or 0)
        end

        return b:sub(index+1, index+1)
    end))
end

-- decoding
function dec(data)
    data = string.gsub(data, '[^'..b..']', '')
    --只允许编码的字符
    return (data:gsub('.', function(x)
        local ret, f = '', (b:find(x)-1)
        for i=6, 1, -1 do 
            ret = ret .. (f % 2^i - f % 2^(i-1) > 0 and '1' or '0')
            --将数字转为二进制
        end
        return ret
    end):gsub('%d%d%d?%d?%d?%d?%d?%d?', function(x)
        if (#x ~= 8) then 
            return '' 
        end
        local c = 0
        for i = 1, 8 do 
            c = c + (x:sub(i,i) == '1' and 2^(8-i) or 0) 
        end
        --每8位作为一个二进制数转变为10进制
        return string.char(c)
        --将ASCII重新转变为字符
    end))
end
```

ps: 注意 f % 2^i - f % 2^(i-1) 适用于判断指定位置是否为1