# code reading
## define of DMA
controller負責I/O Device與Memory之間的資料傳輸，其過程完全不需CPU參與，CPU就有更多時間用在process執行
- DMA大多用在High Speed Block-transfer的I/O Device上
- CPU在設定DMA的controller時，需包含：
    1. I/O command(eg. read/write)
    2. I/O Device physical location for data access
    3. Counter(表示傳輸量有多大)
    4. Memory location
- DMA controller與CPU採Interleaving(交替)使用memory resource，一般稱為"cycle straling"技術
## DMAlib.h

### DMAREQ_FLUSHCOMPLT_PROC

- 當DMA完成所有DMA Request Block 的 SG enrtries時呼叫此函數
- 為IftDmaCopy_FireRequest()其中一個參數
- 呼叫calling DmaCopy_FireRequest()後resource allocator不能重新調用DMA requst block 直到此函數被呼叫
- Input
    1. **DmaReqHandler**:
    2. **UserData**:當呼叫DmaCopy_FireRequest()被儲存
    3. **CRCValue**:CRC-32校驗碼
    4. **status**:

**flush:把資料寫回主記憶體**

### IftDmaCopy_Register
- 當EXECUSERMODEMODS_INITRESALLOC被呼叫時呼叫此函數
- Input
    1. **PostResAvailFnc**: 當IftDmaCopy_AllocRequest()被呼叫但沒有request blocks時，此函數會在系統釋出free block時被呼叫
    2. **MaxRequestNeed**: resource allocator所需的最大block數
    3. **MaxSGPerReq**: 每個DMA requset blcok 所能容納的最大SGList數
    4. **CallBackMTask**

### IftDmaCopy_Register
- 從resource pool分配一個DMA Request Block(由ResourceHandler指定)
- Input
    1. **ResourceHandler**: 由一個暫存程序(IftDmaCopy_Register())回傳

### IftDmaCopy_AddSGLstEntry
- 加上一個SG entry到指定的DMA Request Block
- Input
    1. **DmaReqHandler**: 回傳新增SG entry的block handle
    2. **SrcAdr**: 實體起始位置[0-31]
    3. **SrcAdr_H**: 實體起始位置[32-63]
    4. **DstAdr**: 實體目標位置[0-31]
    5. **DstAdr_H**: 實體起始位置[32-63]
    6. **DatLng**: 資料大小

### IftDmaCopy_AddBlockSGLstEntry
- 發出DMA Request Block到DMA engine執行後續操作
- Input
    1. **DmaReqHandler**: 要發給DMA engine的DMA Reqeust Block handle
    2. **flags**: 
        - IDCFLGS_FREE
        - IDCFLGS_GENINTR
        - IDCFLGS_GENCHECKSUMONLY
        - IDCFLGS_GENCHECKSUMINITVALUE
        - IDCFLGS_COPYANDGENCHECKSUM
    3. **PostCompleteFnc**: 當DMA engine完成此DMA Request Block中德所有SG entries調用此函數
    4. **UserData**: 為PostCompleteFnc之參數
    5. **SeedValue**: CRC-32之初始seed(當IDCFLGS_GENCHECKSUMINITVALUE被set時才有意義)

## IftDmaCopy_FreeRequest
- 釋放由IftDmaCopy_AllocRequest()分配的DMA Request Block
- Input
    1. **DmaReqHandler**: 要被釋放的DMA Reqeust Block handle

## IftDmaCopy_AbortSuspend
- 可選擇性暫停DMA
- 當備用controller失效時被調用
- Input
    1. **DMAChannel**: 欲暫停的DMA channel

## IftDmaCopy_Restart
- 可選擇性重啟DMA
- 當備用controller重啟時被調用
- Input
    1. **DMAChannel**: 欲重啟的DMA channel