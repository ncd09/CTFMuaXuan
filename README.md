# CTFMuaXuan
## Tổng quan
BabyImport là một bài crypto có độ khó trung bình trong khuôn khổ giải CTF Mùa Xuân. Trọng tâm của bài này vẫn nằm ở việc đọc hiểu code mã hoá và tìm ra cơ chế giải mã. Tuy vậy, so với bài trước là Programmer, BabyImport sở hữu code phức tạp hơn một chút. Dưới đây là code mã hoá trong bài:
`
#!/usr/bin/python3
from ducbt import flag

def zero_to_one(bin_):
	str_ = ''
	for i in bin_:
		if i == '0': str_ +='1'
		else: str_ += '0'
	else: print('End.')
	return str_

def convert(flag):
	bin_ = ' '.join(format(ord(x), 'b') for x in flag).replace(' ','1')
	return print("bin_: ",zero_to_one(bin_))

if __name__ == '__main__':
	convert(flag)

#bin_:  01111010010101100011010000111100001001000111100001010110011100100000100000101100001001000001111000100000000110100001011001000000010000000011101000010100000101100100000000101100010000000011011000100000001000100001011001000000001110000011110000011010001101000000010
`
## Phân tích 
Xét hàm `zero_to_one`:
    `
    def zero_to_one(bin_):
	    str_ = ''
	    for i in bin_:
		    if i == '0': str_ +='1'
		    else: str_ += '0'
	    else: print('End.')
	    return str_
    `
Dễ dàng nhận thấy đây chỉ là một hàm hoán đổi kí tự `1` và `0` trong xâu cho trước. Sẽ không có quá nhiều điều cần bàn về hàm này. Nhưng ta hoàn toàn có thể tái sử dụng `zero_to_one` cho việc giải mã sau này.

Xét hàm `convert`:
    `
    def convert(flag):
	    bin_ = ' '.join(format(ord(x), 'b') for x in flag).replace(' ','1')
	    return print("bin_: ",zero_to_one(bin_))
    `
Hàm `format(ord(x), 'b')` sẽ trả về mã ASCII của kí tự `x`dưới dạng nhị phân. Do đó, phần code `' '.join(format(ord(x), 'b') for x in flag)` sẽ ghép mã ASCII của tất cả kí tự trong xâu `flag` về một xâu duy nhất, ngăn cách các kí tự đó bằng khoảng trắng. Sau đó, hàm `replace(' ','1')` sẽ thay thế tất cả khoảng trắng bằng kí tự `1`. Kết quả, xâu bin_ thu được là một xâu chỉ chứa các kí tự `1` và `0` với mã ASCII của tất cả các kí tự trong xâu `flag` và ngăn cách bằng kí tự `1`. Cuối cùng, giá trị trả về của hàm là xâu bin_ nhưng đã qua hoán đổi bởi hàm `zero_to_one`.

## Giải mã
Ở cuối của file babyimport.py ta thấy giá trị của xâu sau mã hoá:
`
#bin_:  01111010010101100011010000111100001001000111100001010110011100100000100000101100001001000001111000100000000110100001011001000000010000000011101000010100000101100100000000101100010000000011011000100000001000100001011001000000001110000011110000011010001101000000010
`
Từ các flag trước, ta có thể đưa ra nhận xét: flag sẽ bao gồm các kí tự như số, chữ cái in hoa/thường, dấu ngoặc nhọn, dấu gạch ngang. Nhưng trong bảng mã ASCII, các kí tự số chỉ được biểu diễn bằng 6 bit trong khi các loại còn lại đều được biểu diễn bằng 7 bit. Vậy ta cần phải khảo sát độ dài xâu sau mã hoá để đoán định số kí tự và sự tồn tại của các kí tự số trong flag. May thay, chiều dài của xâu là 263 = 33 * 7 + 32, vừa vặn đủ chỗ cho 33 kí tự 8 bit và 32 khoảng cách nên ta có thể chắc chắn trong flag không chứa kí tự số (Điều này sẽ khác với flag lần đầu, vốn dĩ chỉ có 261 kí tự nên sẽ xuất hiện 2 kí tự số trong flag). 

Khi đã nắm được cấu trúc của xâu, các bước giải mã được thực hiện như sau:
- Trước hết đảo ngược xâu mã với hàm `zero_to_one` có sẵn.
- Thay thế các kí tự `1` ở các vị trí bội của 8 (8,16,24,32,...) bằng khoảng trắng.
- Convert kết quả thu được từ mã nhị phân về ASCII.
Phần code sau sẽ thực hiện 2 bước đầu của việc giải mã. Kết quả thu được hoàn toàn có thể được convert online hoặc xử lý trên chính python thông qua thư viện binascii.
`
def zero_to_one(bin_):
    str_ = ''
    for i in bin_:
	    if i == '0': str_ +='1'
	    else: str_ += '0'
    return str_
def reveal(bin_):
    str_ = ''
    for i in range(len(bin_)):
        if i % 8 == 7: str_ +=' '
        else: str_ += bin_[i]
    else: print('End.')
    return str_
if __name__ == '__main__':                 bin_="01111010010101100011010000111100001001000111100001010110011100100000100000101100001001000001111000100000000110100001011001000000010000000011101000010100000101100100000000101100010000000011011000100000001000100001011001000000001110000011110000011010001101000000010"
print(reveal(zero_to_one(bin_)))
`
Kết quả thu được: `1000010 1010100 1100101 1100001 1101101 1000011 1010100 1000110 1111011 1101001 1101101 1110000 1101111 1110010 1110100 1011111 1011111 1100010 1110101 1110100 1011111 1101001 1011111 1100100 1101111 1101110 1110100 1011111 1100011 1100001 1110010 1100101 1111101`
Ở đây mình convert trên trang rapidtables.com, thu được flag như sau: BTeamCTF{import__but_i_dont_care}

## Đánh giá
Nói chung, BabyImport không phải là một bài crypto khó, tuy nhiên đó là với trường hợp số kí tự "đẹp" như flag sau. Kể cả với flag đầu gồm 2 kí tự số, chỉ cần giải mã dần từ đầu để tìm các chỗ "bất thường" trong flag và chỉnh sửa số bit tại vị trí đó là có thể thu được kết quả đúng. Việc giải mã chỉ thực sự trở nên phức tạp khi lượng kí tự số xấp xỉ lượng kí tự chữ hay dấu gạch ngang, khi đó sẽ nảy sinh rất nhiều trường hợp cần đoán định. 
