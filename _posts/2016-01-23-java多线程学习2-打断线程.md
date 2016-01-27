# Java多线程学习2-打断线程&等待线程的终止
##打断线程demo
<pre>
/**
 * Created by liang.he on 16/1/12.
 * 测试打断一个线程
 */
public class TestInterruptThread extends Thread{

	private boolean flag = true;

	@Override
	public void run() {
		while (true) {
			System.out.println("i'm still running!");
			if (isInterrupted()) {
				System.out.println("The Prime Generator has been Interrupted");
				return;
			}
		}
	}

	public static void main(String[] args ) {
		TestInterruptThread task = new TestInterruptThread();
		task.start();
		try {
			Thread.sleep(5);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		task.interrupt();
	}
}
</pre>

## 等待线程的终止

调用线程的join()方法。