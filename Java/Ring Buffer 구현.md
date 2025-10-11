# Ring Buffer

## 링 버퍼란
**링 버퍼**(Ring Buffer)는 순환 버퍼, 원형 큐라고도 부른다. 

링 버퍼는 고정된 크기의 배열을 사용하여 데이터를 순환 방식으로 저장하는 자료 구조이다.<br>
배열의 끝에 도달하면 다시 처음으로 돌아가 데이터를 덮어쓰는 방식으로 동작한다.

<img width="437" height="144" alt="ring-buffer-1" src="https://github.com/user-attachments/assets/8b346b9f-e20e-48b0-abbb-c1f9f3dc7152" />

<img width="437" height="144" alt="ring-buffer-2" src="https://github.com/user-attachments/assets/70f58e2c-b47f-465b-b322-1232ac1b0e70" />

### 장점
- **빠른 속도**: 읽기와 쓰기 연산이 모두 O(1)로 동작한다.
- **메모리 효율성**: 고정된 크기의 배열을 재사용하므로 GC 부하가 거의 없다.

## 구현 방식
아래 구현에서는 다음 세가지 정보를 관리하는 방식을 사용했다.
- `readPos`: 읽기 위치
- `writePos`: 쓰기 위치
- `flipped`: 쓰기 위치가 뒤집혔는지 여부 (쓰기 위치가 배열의 처음으로 넘어가서 읽기 위치보다 작은 경우 true)

## 구현 예시
```java
public class RingBuffer {

    private Object[] elements = null;

    private int capacity = 0;
    private int writePos = 0;
    private int readPos = 0;
    private boolean flipped = false;

    RingBuffer(int capacity) {
        this.capacity = capacity;
        this.elements = new Object[this.capacity];
    }

    public boolean offer(Object data) {
        if (isFull()) {
            return false; // Buffer is full
        }

        elements[writePos] = data;
        writePos++;
        if (writePos >= capacity) {
            writePos = 0;
            flipped = true;
        }
        return true;
    }

    public Object poll() {
        if (isEmpty()) {
            return null; // Buffer is empty
        }

        Object data = elements[readPos];
        readPos++;
        if (readPos >= capacity) {
            readPos = 0;
            flipped = false;
        }
        return data;
    }

    public boolean isEmpty() {
        return !flipped && writePos == readPos;
    }

    public boolean isFull() {
        return flipped && writePos == readPos;
    }

    public int size() {
        if (!flipped) {
            return writePos - readPos;
        } else {
            return capacity - (readPos - writePos);
        }
    }
}
```

---
**Reference**
- https://en.wikipedia.org/wiki/Circular_buffer
- https://jenkov.com/tutorials/java-performance/ring-buffer.html
