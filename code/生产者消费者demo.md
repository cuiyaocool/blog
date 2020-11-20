写一个生产者消费者的demo：
- notify 和 wait 要在 synchronized 内调用 
- 锁谁则等谁且释放谁， 如果锁a, 等待时需要a.wait， 释放时需要a.notify

static int b 不需要是volatile是由于锁的性质，会保证修改后把值写入主存

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class ProduceConsumer {
    static int b = 0;
    public static void main(String[] args) {
        Object a = new Object();
        Random r = new Random();

        final List<Integer> list = new ArrayList<>(1);
        Thread producer = new Thread(new Runnable() {
            @Override
            public void run() {
                while (b < 10) {
                    synchronized (a) {
                        while (list.size() != 0) {
                            try {
                                a.wait();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                        list.add(r.nextInt());
                        System.out.println("produce : " + list.get(0));
                        a.notify();
                    }
                }

            }
        });
        Thread consumer = new Thread(new Runnable() {
            @Override
            public void run() {
                while (b < 10) {
                    synchronized (a) {
                        while (list.isEmpty()) {
                            try {
                                a.wait();
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                        b++;
                        System.out.println("consumer : " + list.remove(0));
                        a.notify();
                    }

                }

            }
        });

        producer.start();
        consumer.start();

    }
}

```