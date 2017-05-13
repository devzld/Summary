//装备的接口  

	/**
	 * 装备的接口
	 * 
	 * @author lingdong
	 *
	 */
	public interface IEquip {
	
		/**
		 * 计算攻击力
		 * 
		 * @return
		 */
		public int caculateAttack();
		
		/**
		 * 装备的描述
		 * 
		 * @return
		 */
		public String description();
	}
//装备：武器

	/**
	 * 武器
	 * 攻击力20
	 * 
	 * @author lingdong
	 *
	 */
	public class ArmEquip implements IEquip{
	
		@Override
		public int caculateAttack() {
			
			return 20;
		}
	
		@Override
		public String description() {
			// TODO Auto-generated method stub
			return "屠龙刀";
		}
	
	}
//装备：戒指

	/**
	 * 戒指
	 * 攻击力 5
	 * @author lingdong
	 *
	 */
	public class RingEquip implements IEquip {
	
		@Override
		public int caculateAttack() {
			
			return 5;
		}
	
		@Override
		public String description() {
			
			return "圣战戒指";
		}
	
	}
//装备：护腕

	/**
	 * 护腕
	 * 攻击力 5
	 * @author lingdong
	 *
	 */
	public class WristEquip implements IEquip {
	
		@Override
		public int caculateAttack() {
			
			return 5;
		}
	
		@Override
		public String description() {
			
			return "圣战护腕";
		}
	
	}
//装备：鞋子

	/**
	 * 鞋子
	 * 攻击力 5
	 * @author lingdong
	 *
	 */
	public class ShoeEquip implements IEquip {
	
		@Override
		public int caculateAttack() {
			
			return 5;
		}
	
		@Override
		public String description() {
			
			return "圣战靴子";
		}
	
	}
//装饰者接口

	public interface IEquipDecorator extends IEquip {
	
	}
//装饰品：蓝宝石

	/**
	 * 蓝宝石装饰品
	 * 每颗攻击力+5
	 * @author lingdong
	 *
	 */
	public class BlueGemDecorator implements IEquipDecorator {
	
		/**
		 * 每个装饰品维护一个装备
		 */
		private IEquip equip;
		
		public BlueGemDecorator(IEquip equip) {
			super();
			this.equip = equip;
		}
	
		@Override
		public int caculateAttack() {
			
			return 5+equip.caculateAttack();
		}
	
		@Override
		public String description() {
			
			return equip.description()+"+ 蓝宝石";
		}
	
	}
//装饰品：黄宝石

	/**
	 * 黄宝石装饰品
	 * 攻击力+10
	 * @author lingdong
	 *
	 */
	public class YellowGemDecorator implements IEquipDecorator {
	
		/**
		 * 每个装饰品维护一个装备
		 */
		private IEquip equip;
		
		
		
		public YellowGemDecorator(IEquip equip) {
			super();
			this.equip = equip;
		}
	
		@Override
		public int caculateAttack() {
			
			return 10 + equip.caculateAttack();
		}
	
		@Override
		public String description() {
			
			return equip.description() + "+ 黄宝石";
		}
	
	}
//装饰品：红宝石

	/**
	 * 红宝石装饰 
	 * 每颗攻击力+15
	 * @author lingdong
	 *
	 */
	public class RedGemDecorator implements IEquipDecorator {
	
		/**
		 * 每个装饰品维护一个装备
		 */
		private IEquip equip;
		
		public RedGemDecorator(IEquip equip) {
			super();
			this.equip = equip;
		}
	
		@Override
		public int caculateAttack() {
			
			return 15 + equip.caculateAttack();
		}
	
		@Override
		public String description() {
			// TODO Auto-generated method stub
			return equip.description()+"+ 红宝石";
		}
	
	}
//装饰者模式测试类

	/**
	 * 装饰者模式测试
	 * @author lingdong
	 *
	 */
	public class Test {
	
		public static void main(String[] args) {
			System.out.println("一个镶嵌2颗红宝石，1颗蓝宝石的靴子");
			IEquip equip = new RedGemDecorator(new RedGemDecorator(new BlueGemDecorator(new ShoeEquip())));
			System.out.println("攻击力 ："+equip.caculateAttack());
			System.out.println("描述 ："+equip.description());
			System.out.println("------");
			
			System.out.println("一个镶嵌1颗红宝石，1颗蓝宝石，1颗黄宝石的武器");
			equip = new RedGemDecorator(new BlueGemDecorator(new YellowGemDecorator(new ArmEquip())));
			System.out.println("攻击力 ："+equip.caculateAttack());
			System.out.println("描述 ："+equip.description());
			System.out.println("------");
			
		}
	
	}    
