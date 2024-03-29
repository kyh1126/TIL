# Chapter 13. 그렇다면 뺄셈은 어떨까요?

- 뺄셈에서는 자리올림수(carry)가 발생하는 것이 아니라 빌림수(borrow)가 발생

### 빌림 없이 뺄셈을 처리

---

- 253 - 176 = 77
- 9의 보수(complement): 9로 이루어진 숫자 열에서 어떤 값을 뺀 결과
    - 1️⃣ 빼는 수에 대한 9의 보수를 구한 다음
        - 9의 보수: 999 - 176 = 823
    - 2️⃣ 빼어지는 수에 더한다.
        - 253 + 823 = 1076
    - 3️⃣ 1을 더한 다음에 1000을 뺀다.
        - 1076 + 1 - 1000 = 77

👉

- 1️⃣ 빼는 수를 11111111에서 뺀다.
    - 십진수로는 255
    - 11111111 - 10110000 = 01001111
- 2️⃣ 빼는 수에 대한 1의 보수와 빼어지는 수를 더한다.
    - 11111101 + 01001111 = 101001100
- 3️⃣ 결과에 1을 더한다.
    - 101001100 + 1 = 101001101
- 4️⃣ 100000000을 뺀다.
    - 십진수로는 256
    - 101001101 - 100000000 = 1001101 (77)

<aside>
💡 1의 보수는 비트 단위의 반전을 통하여 얻을 수 있다.(인버터)

</aside>

### 오버플로/언더플로

---

- 여덟 개의 전구로
    - 오버플로: 덧셈의 결과가 255보다 큰 경우
    - 언더플로: 뺄셈의 결과가 음수인 경우

### 부호 비트

---

- 부호 있는 숫자를 나타낼 때 가장 왼쪽에 있는 비트 (보통 MSB, most significant bit)
- 1이 되면 음수를 나타내고 0인 경우에는 양수를 나타낸다.
