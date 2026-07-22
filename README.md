# yule-pos
class SalonCheckoutSystem:
    """
    營業結帳與技師輪牌計算核心模組
    適用於：悠楽 / 樂來姊妹店跨店與自營帳務邏輯
    """
    
    SHEET_FEE = 200  # 床單費 (シーツ代)
    NAME_FEE = 200   # 技師指名費

    @classmethod
    def calculate_order(cls, order):
        """
        計算單筆訂單的明細與費用
        
        :param order: dict 包含以下鍵值
            - basePrice: int, 課程基礎含稅價格 (税込)
            - hasDiscount: bool, 是否享有割引（若有割引則免收床單費）
            - isNamed: bool, 是否指定技師
            - isLailaiPayment: bool, 是否為「樂來收款」（由樂來收款，我方不記帳）
        :return: dict 計算後的帳務與輪牌指示
        """
        # 1. 計算床單費：有割引則免收，否則預設收 200 円
        sheet_fee = 0 if order.get("hasDiscount", False) else cls.SHEET_FEE

        # 2. 計算指名費：有指定技師則收 200 円
        name_fee = cls.NAME_FEE if order.get("isNamed", False) else 0

        # 3. 總金額計算
        total_amount = order.get("basePrice", 0) + sheet_fee + name_fee

        # 4. 帳務歸屬與跳牌邏輯判定
        if order.get("isLailaiPayment", False):
            accounting_record = {
                "storeRevenue": 0,
                "note": "樂來收款，我方不計入帳務 (No Accounting Record)"
            }
        else:
            accounting_record = {
                "storeRevenue": total_amount,
                "note": "正常自營帳務"
            }

        return {
            "basePrice": order.get("basePrice", 0),
            "sheetFee": sheet_fee,
            "nameFee": name_fee,
            "finalPayable": total_amount,
            "accounting": accounting_record,
            # 核心要求：不管哪邊收款，只要是我方技師做，就必須觸發跳牌
            "triggerRotation": True, 
            "message": "計算完成，技師已成功觸發輪牌（跳牌）"
        }


# ==========================================
# 執行測試與驗證
# ==========================================
if __name__ == "__main__":
    
    # 測試範例 1：一般自營客（無折扣、有指定技師）
    normal_order = {
        "basePrice": 7980,
        "hasDiscount": False,
        "isNamed": True,
        "isLailaiPayment": False
    }

    print("--- 測試範例 1：一般自營客 ---")
    result_1 = SalonCheckoutSystem.calculate_order(normal_order)
    for key, value in result_1.items():
        print(f"{key}: {value}")

    print("\n" + "="*40 + "\n")

    # 測試範例 2：樂來收款訂單（我方技師支援，不計我方帳，但要跳牌）
    lailai_support_order = {
        "basePrice": 6000,
        "hasDiscount": True,  # 享有割引，故無床單費
        "isNamed": False,
        "isLailaiPayment": True  # 關鍵：樂來收款
    }

    print("--- 測試範例 2：樂來支援單（不計帳但跳牌）---")
    result_2 = SalonCheckoutSystem.calculate_order(lailai_support_order)
    for key, value in result_2.items():
        print(f"{key}: {value}")
