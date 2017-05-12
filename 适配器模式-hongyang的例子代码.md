//手机类 

	/**
	 * 手机
	 *
	 */
	public class Mobile {	
		/**
		 * 充电
		 */
		public void inputPower(V5Power v5Power) {
			int provideV5Power = v5Power.provideV5Power();
			System.out.println("手机(客户端)：我需要5v电压充电，现在是-->"+provideV5Power+"V");
		}
	}

//提供5v电压的接口

	/**
	 * 提供5v电压的一个接口
	 *
	 */
	public interface V5Power {
		
		public int provideV5Power();
	}
//家用220交流电

	/**
	 * 家用220交流电
	 *
	 */
	public class V220Power {
	
		public int privideV220Power(){
			
			System.out.println("我提供220v交流电压。");
			return 220;
		}
	}
//适配器

	/**
	 * 适配器，把220v电压变成5v
	 *
	 */
	public class V5PowerAdapter implements V5Power{
		
		/**
		 * 组合方式
		 */
		private V220Power v220Power;
		
		public V5PowerAdapter(V220Power v220Power) {
			this.v220Power = v220Power;
		}
	
		@Override
		public int provideV5Power() {
			int power = v220Power.privideV220Power();
			//power经过各种操作
			System.out.println("适配器：我悄悄适配了电压");
			
			return 5;
		}
	
	}
//测试类

	public class Test {
	
		public static void main(String[] args) {
			Mobile mobile = new Mobile();
			V5Power v5Power = new V5PowerAdapter(new V220Power());
			mobile.inputPower(v5Power);
		}
	}
