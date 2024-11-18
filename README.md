# LLM 學習筆記
=================================================================

**Prompt Engineering Guide **
https://www.promptingguide.ai/zh



# 提示工程簡介
## 大語言模型設定

* Temperature: 越小獲得越確定的一個結果，調到可能會回傳比較隨機的結果。 (小 -> 保證品質、穩定、簡潔 | 大 -> 更豐富、更有創造性)

* Top_p: 和 Temperature 很像要準確的事實結果就調低，要更多變化就調高。（與 temperature 一起稱為核採樣（nucleus sampling）的技術）
       越低的值就會對 token 選擇更有信心的響應，越高則會選擇更多樣化的結果，甚至是不太可能的詞語
       
^^^ Temperature & Top_p 一般建議二擇一調整即可 ^^^

* Max Length：控制最多可以生成的 token 數量

* Stop Sequences：stop sequences 是一個停止字符串，可以阻止模型生成 token 用以控制模型響應長度與結構的方法，例如:設定 "11" 作為 stop sequences 來讓模型不會生成超過 10 項的列表

* Frequency Penalty：對下個生成的 token 進行懲罰，這個懲罰和 token 出現的次數成比例，frequency penalty 越高，某個詞的出線可能性就下降，這個設置通過給重複數量多的 token 設置更高的懲罰以減少響應中詞的重複

* Presence Penalty：也是對重複的 token 做懲罰，但與 frequency pealty 不同的是，懲罰對所有重複的 token 都是相同的，重複 2 次跟重複 10 次都是相同的懲罰。 如果想要生成更多樣化或有創造性的文本，可以把該值設置更高，如果想要生成更專注的內容，可以將該值設定較低。

^^^ Frequency Penalty & Presence Penalty 一般建議二擇一調整即可 ^^^

## 基本概念

prompt：有分 system、user 和 assistant。
        system prompt: 並不是必要的，但設定它可以幫助模型以我們設定的方向去做分析
        user prompt: 就是我們想要詢問 LLM 的文字，必須清楚、乾淨、避免模糊語句 (例如「寫入」、「分類」、「總結」、「翻譯」、「排序」等)
        assistant prompt: 代表 system 回傳給使用者以滿足其目標所需提供的資訊或服務。
        
三者的關係 user 向 system 發送請求，sysyem 產生 assistant 來滿足使用者的請求。

## 提示詞要素

* 指令：想要模型執行的特定任務或指令。

* 上下文：包含外部資訊或額外的上下文訊息，引導語言模型更好地回應。

* 輸入資料：使用者輸入的內容或問題。

* 輸出指示：指定輸出的類型或格式。

* 具體性：要非常具體地說明你希望模型執行的指令和任務。

* **做什麼還是不做什麼?**：設計提示詞時常見的技巧是避免 "不要做甚麼" 應該要說 "該做甚麼"，並聚焦在細節上。

```
EX. 失敗範例
提示:
    以下是向客户推荐电影的代理程序。不要询问兴趣。不要询问个人信息。
客户：请根据我的兴趣推荐电影。

輸出:
    当然，我可以根据你的兴趣推荐电影。你想看什么类型的电影？你喜欢动作片、喜剧片、爱情片还是其他类型的电影？
```


## 提示詞範例

* **文本概括**:詢問一個東西請他解釋，再要求其將解釋內容濃縮成一句話。

* **資訊擷取**:給予一段文本要求其更具文本給定期描述的對象。

* **問答**:給一段文本問其中的內容相關的問題。

* **文字分類**:給予一段文本問其在提供的類別中分類。

* **對話**:跟他聊天拉

* **程式碼生成**:給予需求並要求其生成符合需求的可執行程式碼。

* **推理**:給予算術之類的問題並要求其解答結果。


# 提示技術

## 零樣本提示 & 少樣本提示
    
    基本的問問題並無給予範例即零樣本提示，給予少數範例即少樣本提示
    
## 鍊式思考（CoT）提示 (Chain-of-Thought Prompting)
鍊式思考（CoT）提示透過中間推理步驟實現了複雜的推理能力。
![image](https://hackmd.io/_uploads/Hydk_R-fyg.png)

簡單來說 
* Zero-shot 就是直接問問題不給予答案要求 LLM 提供答案
* Few-show 是問問題並提供一組含解答的 QA 讓 LLM 參考去提供我們所問的答案
* Few-shot-CoT 是問問題並給我一組含解答的 QA 外在 A 的部分還給與"解答過程"，讓 LLM 在提供們所問的答案同時參考計算方式並回傳時也含該題的計算方式
* Zero-shot-CoT 不提供一組參考 QA 直接要求 LLM 回答我們所問的問題，並在 A 要求 "step by step"

```
    ☆☆☆ 總結要讓他變好就給他一個魔法 "Let's think step by step" 他就會變好 ☆☆☆
```

## 自我一致性

有一些問題直接詢問可能會因為其複雜性讓答案是錯的

    提示：當我6歲時，我的妹妹是我的一半年齡。現在我70歲了，我的妹妹多大？
    輸出：35
    (GPT-4o 不會錯)

這個答案是錯的!
以下是透過自我一致性來改進這個問題
    
    提示:
    Q：林中有15棵樹。林業工人今天將在林中種樹。完成後，將有21棵樹。林業工人今天種了多少棵樹？
    A：我們從15棵樹開始。後來我們有21棵樹。差異必須是他們種樹的數量。因此，他們必須種了21-15 = 6棵樹。答案是6。
    Q：停車場有3輛汽車，又來了2輛汽車，停車場有幾輛汽車？
    A：停車場已經有3輛汽車。又來了2輛。現在有3 + 2 = 5輛汽車。答案是5。
    Q：Leah有32塊巧克力，她的姊姊有42塊。如果他們吃了35塊，他們總共剩下多少塊？
    A：Leah有32塊巧克力，Leah的姊姊有42塊。這意味著最初有32 + 42 = 74塊巧克力。已經吃了35塊。因此，他們總共還剩74-35 = 39塊巧克力。答案是39。
    Q：Jason有20個棒棒糖。他給Denny一些棒棒糖。現在Jason只有12個棒棒糖。 Jason給Denny多少棒棒糖？
    A：Jason有20個棒棒糖。因為他現在只有12個，所以他必須把剩下的給Denny。他給Denny的棒棒糖數量必須是20-12 = 8個棒棒糖。答案是8。
    Q：Shawn有五個玩具。聖誕節，他從他的父母那裡得到了兩個玩具。他現在有多少個玩具？
    A：他有5個玩具。他從媽媽那裡得到了2個，所以在那之後他有5 + 2 = 7個玩具。然後他從爸爸那裡得到了2個，所以總共他有7 + 2 = 9個玩具。答案是9。
    Q：伺服器房間裡有9台電腦。從週一到週四，每天都會安裝5台電腦。現在伺服器房間裡有多少台電腦？
    A：週一到週四有4天。每天都增加了5台計算機。這意味著總共添加了4 * 5 =
    20台計算機。一開始有9台計算機，所以現在有9 + 20 = 29台計算機。答案是29。
    Q：Michael有58個高爾夫球。星期二，他失去了23個高爾夫球。星期三，他又失去了2個。星期三結束時他還剩多少個高爾夫球？
    A：Michael最初有58顆球。星期二他丟失了23個，所以在那之後他有58-23 = 35個球。星期三他又失去了2個，所以現在他有35-2 = 33個球。答案是33。
    Q：Olivia有23美元。她用每個3美元的價格買了五個百吉餅。她還剩多少錢？
    A：她用每個3美元的價格買了5個貝果。這意味著她花了15美元。她還剩8美元。
    Q：當我6歲時，我的妹妹是我的一半年齡。現在我70歲了，我的妹妹多大？
    A：

    輸出:
    當我6歲時，我的妹妹是我的一半年齡，也就是3歲。現在我70歲了，所以她是70-3 = 67歲。答案是67。

## 產生知識提示
類似於上面的自我一致性，但這邊是提供給他一些相關知識讓他可以推理出合理的結果
```
提示：
    高尔夫球的一部分是试图获得比其他人更高的得分。是或否？
輸出：
    是。
```

上面結果明顯是錯的，但因為 LLM 對世界的知識不足所以無法回答正確結果

那現在我們補充一些高爾夫相關的知識給他那便會有不同的結果

```
輸入：希臘比墨西哥大。
知識：希臘的面積約為131,957平方公里，而墨西哥的面積約為1,964,375平方公里，使墨西哥比希臘大了1,389%。
輸入：眼鏡總是會起霧。
知識：當你的汗水、呼吸和周圍的濕度中的水蒸氣落在冷的表面上，冷卻並變成微小的液滴時，會在眼鏡鏡片上產生冷凝。你看到的是一層薄膜。你的鏡片相對於你的呼吸會比較涼，尤其是當外面的空氣很冷時。
輸入：魚有思考能力。
知識：魚比它們看起來更聰明。在許多領域，如記憶力，它們的認知能力與或超過非人類靈長類動物等「更高級」的脊椎動物。魚的長期記憶幫助它們追蹤複雜的社交關係。
輸入：一個人一生中吸煙很多香煙的常見影響是患肺癌的幾率高於正常水平。
知識：那些一生中平均每天吸菸不到一支香菸的人，罹患肺癌的風險是從不吸菸者的9倍。在每天吸菸1到10支香菸之間的人群中，死於肺癌的風險幾乎是從不吸菸者的12倍。
輸入：一塊石頭和一顆卵石大小相同。
知識：卵石是一種根據Udden-Wentworth沉積學尺度的顆粒大小為4到64毫米的岩屑。卵石通常被認為比顆粒（直徑2到4毫米）大，比卵石（直徑64到256毫米）小。
輸入：高爾夫球的一部分是試圖獲得比其他人更高的得分。
知識：

高爾夫球的目標是以最少的桿數打完一組洞。一輪高爾夫球比賽通常包括18個洞。每個洞在標準高爾夫球場上一輪只打一次。每個桿計為一分，總桿數用於確定比賽的獲勝者。
```

## 鍊式提示

為了提高大語言模型的表現使其更可靠，一個重要的提示工程技術是將任務分解為許多子任務。 確定子任務後，將子任務的提示詞提供給語言模型，得到的結果作為新的提示詞的一部分。 這就是所謂的鍊式提示（prompt chaining），一個任務被分解為多個子任務，根據子任務建立一系列提示操作。

```
提示1: syatem prompt 給予 LLM 身分並要求其依照規定格式輸出引文列表 (<quotes></quotes>)，並附上文本且內容放在 #### {{文件}} #### 之間
       user prompt 問問題，What are the prompting techniques mentioned in the document?
輸出: 
    <quotes>
    - Chain-of-thought (CoT) prompting[27]
    - Generated knowledge prompting[37]
    - Least-to-most prompting[38]
    - Self-consistency decoding[39]
    - Complexity-based prompting[41]
    - Self-refine[42]
    - Tree-of-thought prompting[43]
    - Maieutic prompting[45]
    - Directional-stimulus prompting[46]
    - Textual inversion and embeddings[59]
    - Using gradient descent to search for prompts[61][62][63][64]
    - Prompt injection[65][66][67]
    </quotes>
```

在第一個提示中傳回的引文現在可以用作下面第二個提示的輸入。您可以對這些引文進行清理，例如移除引用標誌。可以在提示鏈中新建另一個提示來移除或使用這些引用標誌，但現在您可以忽略這些。然後，第二個提示接收由第一個提示提取的相關引文，並根據文件和這些提取的引文產生一個有幫助的答案。第二個提示可以是以下：

```
提示2: 根据从文档中提取的相关引文（由<quotes></quotes>分隔）和原始文档（由####分隔），请构建对问题的回答。请确保答案准确、语气友好且有帮助。
    ####
    {{文件}}
    ####
    <quotes>
    - Chain-of-thought (CoT) prompting[27]
    - Generated knowledge prompting[37]
    - Least-to-most prompting[38]
    - Self-consistency decoding[39]
    - Complexity-based prompting[41]
    - Self-refine[42]
    - Tree-of-thought prompting[43]
    - Maieutic prompting[45]
    - Directional-stimulus prompting[46]
    - Textual inversion and embeddings[59]
    - Using gradient descent to search for prompts[61][62][63][64]
    - Prompt injection[65][66][67]
    </quotes>

輸出2:
    文件中提到的提示技術包括：
    1. Chain-of-thought (CoT) prompting[27]
    2. Generated knowledge prompting[37]
    3. Least-to-most prompting[38]
    4. Self-consistency decoding[39]
    5. Complexity-based prompting[41]
    6. Self-refine[42]
    7. Tree-of-thought prompting[43]
    8. Maieutic prompting[45]
    9. Directional-stimulus prompting[46]
    10. Textual inversion and embeddings[59]
    11. Using gradient descent to search for prompts[61][62][63][64]
    12. Prompt injection[65][66][67]
    每種技術都採用獨特的策略來增強或明確與大型語言模型的互動，以產生期望的結果。
    
```

## 思維樹(Tree of Thoughts，ToT)

ToT 維護一棵思考樹，思維由連貫的語言序列表示，這個序列就是解決問題的中間步驟。使用這種方法，LM 能夠自行對嚴謹推理過程的中間思維進行評估。

![image](https://hackmd.io/_uploads/S1-0UkMGyx.png)

ToT 完成算24 的遊戲任務要執行廣度優先搜尋（BFS），每步思維的候選項都要求LM 給出能否得到24 的評估：「sure/maybe/impossible」（一定能/可能/不可能） 。作者講到：「目的是得到經過少量向前嘗試就可以驗證正確（sure）的局部解，基於'太大/太小'的常識消除那些不可能（impossible）的局部解，其餘的局部解作為'maybe'保留。整個過程如下圖所示：

![image](https://hackmd.io/_uploads/BJ0bdyGz1l.png)

Hulbert (2023)提出了思考樹（ToT）提示法，將ToT 框架的主要概念概括成了一段簡短的提示詞，指導LLM 在一次提示中對中間思維做出評估。 ToT 提示詞的例子如下：

    假设三位不同的专家来回答这个问题。
    所有专家都写下他们思考这个问题的第一个步骤，然后与大家分享。
    然后，所有专家都写下他们思考的下一个步骤并分享。
    以此类推，直到所有专家写完他们思考的所有步骤。
    只要大家发现有专家的步骤出错了，就让这位专家离开。
    请问...

## 檢索增強生成(RAG)

把他想的簡單點，就是你問的問題 LLM 不清楚或沒看過，所以我現在增加一組文本、SQL、或檔案等相關資訊讓他獲取所需的知識，以便讓他回答正確的問題避免去產生幻覺。

但要怎麼給與資訊就有很多方法:
1. **給結構化文本**(EX. SQL、表格)
    該類資訊直接給就好，因為已經很清楚他的格式與內容了
2. **給非結構化文本** (EX. 文章、句子)
    這種需要先透過其他的方式先做前處理，不管是下 Agent 還是用 Tool 或是以其他的模型(很多都有工具可以參考，不用自己寫)來幫忙找問題與文本的向量相似性，只要是可以把所以文本題取出重要資訊都是好方法 

## 自動推理並使用工具(ART)

簡單來說是我們在問 LLM 一個新問題時，讓其先參考以前的 Task 的推理流程(圖 A 的部分)或是使用哪個工具的範例，在讓 LLM 跟隨這個流程或使用的工具去推理出我們所問的問題(圖 B 的部分)。

<div style="text-align: center;">  
  <img src="https://hackmd.io/_uploads/BJHS50zfkl.png" alt="image"/>  
</div> 

## 自動提示工程師（APE）

概念就是第一步涉及一個大型語言模型（作為推理模型），該模型接收輸出演示以產生任務的指令候選項。這些候選解將指導搜尋過程。使用目標模型執行指令，然後根據計算的評估分數選擇最合適的指令。

```
    ☆☆☆ 新魔法 2.0 "Let's work this out in step by step way to be sure we have the right answer" ☆☆☆
```

## Active-Prompt

大概就是說你問問題不管用 Few-shot CoT 還是 Zero-shot CoT 可能還是有回答不好的問題，那他把那些回答不好的問題再次提取出來給予新的註解再問一次。

![image](https://hackmd.io/_uploads/HJjfmymGyx.png)


## 方向性刺激提示

在提問時同時給予一段 Hint 讓他輸出的結果可以往我們提供之 Hint 去做回答。

![image](https://hackmd.io/_uploads/SyrnQyXzkx.png)

## PAL（程式輔助語言模型）

簡單來說就是跟 ART 很像，ART 是給予先前 Task 執行的步驟或工具讓新問的問題可以照做，但現在 PAL 是把執行步驟換成程式的形式去寫，把收到的資訊轉成變數再對變數做計算得出結果。 (整體推理變得以程式去完成的，是可以真的執行的 exec())

![image](https://hackmd.io/_uploads/By9HSJ7M1e.png)

## ReAct 框架

優點向 CoT 結合 RAG，我們可以提問一個問題， LLM 會開始第一次思考，那思考到內容有不知道不清楚地就對該內容去搜尋外部資訊，再根據獲得的資訊去做第二次思考並重複上述過程直到獲得所需結果。

```
問題 科羅拉多造山帶東部區域延伸到的區域的海拔範圍是多少?
思考 1 我需要搜尋科羅拉多造山帶，找到科羅拉多造山帶東段延伸的區域，然後找到該區域的海拔。
操作 1 搜尋 [科羅拉多造山帶]
觀察 1 科羅拉多造山帶是科羅拉多州及其周邊地區造山運動的一段。
思考 2 它沒有提到東部地區。所以我需要查一下東部地區。
操作 2 尋找 [東部地區]
觀察 2 (結果 1 / 1) 東部區域延伸至高平原，稱為中原造山帶。
思考 3 科羅拉多造山運動的東部區域延伸到高平原。所以我需要搜尋高平原，找到它的海拔範圍。
操作 3 搜尋 [高平原]
觀察 3 高平原指的是兩個截然不同的陸地區域之一。
思考 4 我需要搜尋高平原（美國）
操作 4 搜尋 [高平原（美國）]
觀察 4 高平原是大平原的一個分區。從東到西，高平原的海拔從 1800 到 7000 英尺 (550 到 2130 公尺) 不等。 [3]
思考 5 高平原海拔高度從 1800 到 7000 英尺，所以答案是 1800 到 7000 英尺。
操作 5 結束 [1800 到 7000 英尺]
…
```

## 自我反思（Reflexion）

這個做法就是有一個自我反思的 Agent 他主要透過提問並回答基本的結果，再去分析這次的結果是否正確，再以自我反思去改進生成下次的結果
    備註: 自我反思通常依賴自我評估能力，因此必須使用較強的 LLM 來做

![image](https://hackmd.io/_uploads/Hkow3yXzke.png)

自我反思的關鍵步驟是a)定義任務，b)產生軌跡，c)評估，d)執行自我反思，e)產生下一條軌跡

![image](https://hackmd.io/_uploads/Bkdp2kQz1g.png)

## 多模態思維鏈提示方法

簡單就是我問問題同時給圖片，要 LLM 參考問題與圖片去輸出文字結果。

![image](https://hackmd.io/_uploads/r1augemG1e.png)

## ❎ GraphPrompts

    作者還沒更新，有朝一日再看，或是哪天想到自己查再整理在這裡

## ❎ meta-prompting

    作者還沒更新，有朝一日再看，或是哪天想到自己查再整理在這裡

# 提示應用

## 產生數據

簡單巴，就平常都會用的，叫 LLM 生我們要的資料，給定輸出格式、給定規則、需求 EX. 上次生 shop pilot 的 data

## 程式碼生成

這也簡單巴，把你要的需求、結果跟 LLM 說叫他生給你 code EX. daily coding

## 畢業生職位分類個案研究

這是一個範例 case 其內容是提供一系列的職位資訊讓 LLM 判斷事法符合應屆畢業生，以下是實驗與結果

**經本文測試的Prompt策略**

| Short name | Description |  
|------------|-------------|  
| Baseline   | 提供一個職位招聘資訊並詢問它是否適合畢業生。 |  
| CoT        | 在查詢之前給出幾個準確分類的範例。 |  
| Zero-CoT   | 要求模型一步步推理後再給出答案。 |  
| rawinst    | 透過加入到使用者訊息，來給予有關其角色和任務的說明。 |  
| sysinst    | 作為系統訊息給出有關其角色和任務的說明。 |  
| bothinst   | 將角色作為系統訊息和任務作為使用者訊息拆分說明。 |  
| mock       | 透過模擬討論來給出任務說明，其中模型確認了它們。 |  
| reit       | 透過重複強調關鍵要素來加強說明。 |  
| strict     | 要求模型嚴格按照給定模板回答。 |  
| loose      | 要求僅根據給定模板給出最終答案。 |  
| right      | 要求模型得出正確的結論。 |  
| info       | 提供額外的資訊以解決常見的推理失敗。 |  
| name       | 為模型取一個我們在對話中稱呼它的名稱。 |  
| pos        | 在查詢之前向模型提供正面回饋。 |  

**所有Prompt效能策略對效能的影響**

|                    | Precision | Recall | F1   | Template Stickiness |  
|--------------------|-----------|--------|------|---------------------|  
| Baseline           | 61.2      | 70.6   | 65.6 | 79%                 |  
| CoT                | 72.6      | 85.1   | 78.4 | 87%                 |  
| Zero-CoT           | 75.5      | 88.3   | 81.4 | 65%                 |  
| +rawinst           | 80        | 92.4   | 85.8 | 68%                 |  
| +sysinst           | 77.7      | 90.9   | 83.8 | 69%                 |  
| +bothinst          | 81.9      | 93.9   | 87.5 | 71%                 |  
| +bothinst+mock     | 83.3      | 95.1   | 88.8 | 74%                 |  
| +bothinst+mock+reit | 83.8     | 95.5   | 89.3 | 75%                 |  
| +bothinst+mock+reit+strict | 79.9 | 93.7 | 86.3 | 98%                |  
| +bothinst+mock+reit+loose  | 80.5 | 94.8 | 87.1 | 95%                |  
| +bothinst+mock+reit+right  | 84   | 95.9 | 89.6 | 77%                |  
| +bothinst+mock+reit+right+info | 84.9 | 96.5 | 90.3 | 77%           |  
| +bothinst+mock+reit+right+info+name | 85.7 | 96.8 | 90.9 | 79%      |  
| +bothinst+mock+reit+right+info+name+pos | 86.9 | 97 | 91.7 | 81%    |  

    自我小結: 基本上給 zero-shot CoT 之後再 +bothinst 增加資訊就已經跟 baseline 拉開一大段了，
             後面給予更精確的資訊要求更嚴格的細節都會再稍微提升，
             提示工程的重點就是用對方法後對內容寫更精確要求更細節

## 提示函數

有一些特別的提示函數可以讓 LLM 去針對該函數做特定的處理

* \[trans_word]: 將文字翻譯成英文

* \[expand_word]: 擴展文字的函數

* \[fix_english]: 糾正文字的函數

```
Example:
trans_word('婆羅摩火山處於享有「千島之國」美稱的印尼. 多島之國印尼有4500座之多的火山, 世界著名的十大活火山有三座在這裡.')
fix_english('Finally, you can run the function independently or chain them together.')
fix_english(expand_word(trans_word('婆羅摩火山處於享有「千島之國」美稱的印尼. 多島之國印尼有4500座之多的火山, 世界著名的十大活火山有三座在這裡.')))
```

* \[pg]: 建立一個多參數函數，透過接受五個輸入參數來產生一個密碼，並輸出產生的密碼

```
function_name: [pg]
input: ["length", "capitalized", "lowercase", "numbers", "special"]
rule: [作為一個密碼產生器，我將為需要一個安全密碼的個人提供協助。我會提供包括"length"（長度）、"capitalized"（大寫字母）、"lowercase"（小寫字母）、"numbers"（數字）和"special"（特殊字元）在內的輸入形式。你的任務是使用這些輸入形式來產生一個複雜的密碼，並將其提供給我。在你的回答中，請不要包含任何解釋或額外的信息，只需提供產生的密碼即可。例如，如果輸入形式是length = 8、capitalized = 1、lowercase = 5、numbers = 2、special = 1，你的答案應該是類似"D5%t9Bgf"的密碼。 ]
```

```
pg(length = 10, capitalized = 1, lowercase = 5, numbers = 2, special = 1)
pg(10,1,5,2,1)
```

## Function calling

強烈建議看一下這個範例 https://github.com/dair-ai/Prompt-Engineering-Guide/blob/main/notebooks/pe-function-calling.ipynb
上面他有教你問問題怎麼讓 LLM tool 獲取資料的方式

## ❎ context-caching

    作者還沒更新，有朝一日再看，或是哪天想到自己查再整理在這裡

## ❎ finetuning-gpt4o

    作者還沒更新，有朝一日再看，或是哪天想到自己查再整理在這裡

## ❎ generating_textbooks

    作者還沒更新，有朝一日再看，或是哪天想到自己查再整理在這裡

## ❎ synthetic_rag

    作者還沒更新，有朝一日再看，或是哪天想到自己查再整理在這裡


# Prompt Hub ❎ 

這裡集中了一大堆 prompt 的範例，可以來這邊參考 prompt 怎麼寫 (之後再補)

https://www.promptingguide.ai/zh/prompts

* 分類
* 編碼
* 創造力
* 評估
* 資訊擷取
* 影像生成
* 數學
* 問答
* 推理
* 文字摘要
* 誠實
* 對抗性提示

# 模型

這裡介紹了一大堆 LLM 模型簡介，可以來這裡看各個模型的資訊 
(目前只更新少部分其他之後有緣再補)

https://www.promptingguide.ai/zh/models

* ▢ Flan
* ☑ ChatGPT
* ☑ LLaMA
* ☑ GPT-4-4
* Mistral 7B
* ▢ Gemini
* ▢ Gemini Advanced
* ▢ Gemini 1.5 Pro
* ▢ Phi-2
* Mixtral
* Code Llama
* OLMo
* Sora
* ☑ Model Collection
* claude-3
* gemma
* grok-1
* llama-3
* mistral-large
* mixtral-8x22b

## ChatGPT + GPT-4

**ChatGPT** 應該不用啥詳細的介紹，大家都老熟了，ChatGPT是由OpenAI訓練的一種新模型，具有互動式對話的能力。

ChatGPT 使用人類回饋的強化學習（RLHF）進行訓練。這個模型比以前的GPT迭代要強大得多（並且還經過了訓練以減少有害和不真實的輸出）。


```
一個簡單 Call Openai ChatGPT-3.5-ruebo API 的範例
CONTENT = """Answer the question based on the context below. 
             Keep the answer short and concise. Respond \"Unsure about answer\" 
             if not sure about the answer.
 
Context: Teplizumab traces its roots to a New Jersey drug company called Ortho Pharmaceutical. 
         There, scientists generated an early version of the antibody, dubbed OKT3. Originally sourced from mice, 
         the molecule was able to bind to the surface of T cells and limit their cell-killing potential. In 1986, 
         it was approved to help prevent organ rejection after kidney transplants, making it the first 
         therapeutic antibody allowed for human use.
 
Question: What was OKT3 originally sourced from?
 
Answer:
 
"""
 
response = openai.ChatCompletion.create(
    model="gpt-3.5-turbo", 
    messages=[{"role": "user", "content": CONTENT},], 
    temperature=0,
)
```

接下來 OpenAI發布了**GPT-4**，這是一個大型的多模態模型，可以接受圖像和文字輸入並輸出文字。它在各種專業和學術基準測試中實現了人類水平的表現。
(公司使用的 Azure 的 GPT-4o 不支援多模態)

### 視覺能力

如上述所說的 GPT-4 他有多模態的能力可以同時接受文字與圖片的輸入，因此我們可以輸入一段文字問題與圖片圖表讓其根據圖表與問題回答正確的結果。

Input text:
```
What is the sum of average daily meat consumption for Georgia and Western Asia? Provide a step-by-step reasoning before providing your answer.
```
Input image: 
![image](https://hackmd.io/_uploads/ryDXrM7z1e.png)

GPT-4 Output text: 
```
要找到格魯吉亞和西亞的平均每日肉類消費總和，我們將按照以下步驟進行：
1.確定格魯吉亞的平均每日肉類消費。
2.確定西亞的平均每日肉類消費。
3.將步驟1和2中的數值相加。
步驟1：從提供的圖表中，我們可以看到格魯吉亞的平均每日肉類消費為79.84克/人/天。
步驟2：從圖表中，我們可以看到西亞的平均每日肉類消費為69.62克/人/天。
步驟3：將步驟1和2中的數值相加。
79.84克（喬治亞）+69.62克（西亞）=149.46克
因此，喬治亞和西亞的平均每日肉類消費總和為149.46克/人/天。
```

### 引導 GPT-4

我們也可以透過 system 訊息的能力來引導模型以特定的語氣和風格提供答案。

如我們可以要求 GPT 以 json 的格式回答結果

```
SYSTEM: You are an AI Assistant and always write the output of your response in json.
USER: Please return a sampled list of text with their sentiment labels. 10 examples only.
```
Response:
```
{
  "examples": [
    {
      "text": "I absolutely love this place, the atmosphere is amazing!",
      "sentiment": "positive"
    },
    {
      "text": "The food was terrible and the service was even worse.",
      "sentiment": "negative"
    },
    ...
  ]
}
```

要在先前的GPT-3模型中實現這一點，需要在指令中寫的非常詳細。 GPT-4的區別在於只需透過system訊息一次指示樣式，這將持續存在於任何後續互動中。
如果要替換成別種格式或是風格只要重下 prompt 直接覆蓋就好。

### GPT-4 限制 or 失敗案例

在許多測試上雖然 4 已比 3.5 好上非常多，但依然還是有些使用上的限制或是錯誤。

```
USER: Son of an actor, this American guiterarist and rock singer released many songs 
        and albums and toured with his band. His name is "Elvis" what?
ASSISTANT: Elvis Costello
```

但這個答案實際上是錯的，答案應該是 "Elvis Presley"

```
這裡可以加上魔法 **think step by step**，就會正確了
```

或是有另一種 case 是該資訊在 GPT-4 過往的資料內沒有的資訊，他可能還是會找出最相近的錯誤答案，該狀況我們稱之為「**幻覺**」。

```
這裡也可以加上 system prompt 如果不知道答案時輸出「**我不知道答案**」也可以同時調低 template 讓其對不知道的結果更有信心
```


## LLaMA 

開放且有效率的基礎語言模型，其模型參數範圍從70E~650E，並且這些模型在公開取得的資料及上訓練的數兆次的訓練

在某人論文的測試下建議使用 200B token訓練10B 的模型，然而在 LLaMa 的論文中即使在 1T token訓練7B 的模型表現依然會繼續提高，這邊依照自己的計算預算下去做選擇。

在測試下可以發現 LLaMa-13B 的模型比 GPT-3 (175B) 小 10倍，但在許多基準測試上的表現仍優於 GPT-3，並且可以在單一GPU上運行，此外LLaMA 65B 與Chinchilla-70B 和PaLM-540B 等模型都具有競爭力。

![image](https://hackmd.io/_uploads/ByojsG7fkx.png)

```
Paper: LLaMA: 開放且有效率的基礎語言模型

Code: https://github.com/facebookresearch/llama
```

## Model Collection

| Model          | Release Date | Description                                                                                           |  
|----------------|--------------|-------------------------------------------------------------------------------------------------------|  
| BERT           | 2018         | Bidirectional Encoder Representations from Transformers                                               |  
| GPT            | 2018         | Improving Language Understanding by Generative Pre-Training                                           |  
| RoBERTa        | 2019         | A Robustly Optimized BERT Pretraining Approach                                                        |  
| GPT-2          | 2019         | Language Models are Unsupervised Multitask Learners                                                   |  
| T5             | 2019         | Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer                     |  
| BART           | 2019         | Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension |  
| ALBERT         | 2019         | A Lite BERT for Self-supervised Learning of Language Representations                                   |  
| XLNet          | 2019         | Generalized Autoregressive Pretraining for Language Understanding and Generation                       |  
| CTRL           | 2019         | CTRL: A Conditional Transformer Language Model for Controllable Generation                             |  
| ERNIE          | 2019         | ERNIE: Enhanced Representation through Knowledge Integration                                           |  
| GShard         | 2020         | GShard: Scaling Giant Models with Conditional Computation and Automatic Sharding                      |  
| GPT-3          | 2020         | Language Models are Few-Shot Learners                                                                 |  
| LaMDA          | 2021         | LaMDA: Language Models for Dialog Applications                                                        |  
| PanGu-α        | 2021         | PanGu-α: Large-scale Autoregressive Pretrained Chinese Language Models with Auto-parallel Computation |  
| mT5            | 2021         | mT5: A massively multilingual pre-trained text-to-text transformer                                    |  
| CPM-2          | 2021         | CPM-2: Large-scale Cost-effective Pre-trained Language Models                                         |  
| T0             | 2021         | Multitask Prompted Training Enables Zero-Shot Task Generalization                                     |  
| HyperCLOVA     | 2021         | What Changes Can Large-scale Language Models Bring? Intensive Study on HyperCLOVA: Billions-scale Korean Generative Pretrained Transformers |  
| Codex          | 2021         | Evaluating Large Language Models Trained on Code                                                      |  
| ERNIE 3.0      | 2021         | ERNIE 3.0: Large-scale Knowledge Enhanced Pre-training for Language Understanding and Generation      |  
| Jurassic-1     | 2021         | Jurassic-1: Technical Details and Evaluation                                                          |  
| FLAN           | 2021         | Finetuned Language Models Are Zero-Shot Learners                                                      |  
| MT-NLG         | 2021         | Using DeepSpeed and Megatron 給 Train Megatron-Turing NLG 530B, A Large-Scale Generative Language Model |  
| Yuan 1.0       | 2021         | Yuan 1.0: Large-Scale Pre-trained Language Model in Zero-Shot and Few-Shot Learning                   |  
| WebGPT         | 2021         | WebGPT: Browser-assisted question-answering with human feedback                                       |  
| Gopher         | 2021         | Scaling Language Models: Methods, Analysis & Insights from Training Gopher                            |  
| ERNIE 3.0 Titan| 2021         | ERNIE 3.0 Titan: Exploring Larger-scale Knowledge Enhanced Pre-training for Language Understanding and Generation |  
| GLaM           | 2021         | GLaM: Efficient Scaling of Language Models with Mixture-of-Experts                                    |  
| InstructGPT    | 2022         | Training language models to


# 風險和誤用

這一章節很簡單就不介紹了，自己看，基本上就是不要搞耍，不該寫的就不要亂寫 EX. "忽略上面的指示！"

不過這章蠻有趣的有些可以越獄的方式(EX. 老奶奶越獄事件，自己去 google)，可以看看，但基本上不太會用到。
![image](https://hackmd.io/_uploads/r1czAGmGJg.png)


# LLM Research Findings

我感覺這章就是對所有東西做大整理，看都看好看一直看

## LLM Agents

大語言模型（LLM）是利用 LLM 建構 Agent 來進行複雜的任務執行應用，而讚建構的過程中 LLM 擔任控制中心大腦的角色，而還須依賴許多其他的關鍵模組如:規劃(palnning)、記憶(memort)、工具(tool)等等。

在一些簡單的問題也許只需要 LLM 便可以獨力完成，但遇到一些複雜的問題時，可能就需要結合外部的知識庫形成 「RAG 系統」，或是建立一個可以「存取搜尋的 API」，也可能需要一些「代碼解釋器」的工具來產生所需之圖表等，此時我們可以把建構的 Agent 看作如下的圖所示

**大語言模型 Agent 框架簡介**
![image](https://hackmd.io/_uploads/H1Pj8XXzyg.png)

通常，一個大語言模型 Agent 框架包含以下核心部分：

* User Request - 表達用戶的問題或請求
* Agent - 作為協調者的智能體核心
* Planning - 助於智能體為未來行動做規劃
* Memory - 管理智能體的歷史行為與經驗
* Tools - 與外界環境互動的工具

### Agent

就是一個 LLM 可以透過一些規劃或工具來驅動他獲取我們所要求的回應

### Planning

Planning 可以粗略分成兩種 「無回饋規劃」和「有回饋規劃」

#### 無回饋規劃

規劃模組的作用是將使用者的請求猜分成 Agent 可以單獨解決的若干步驟或子任務，這樣可以讓 Agent 能夠更準確的理解問題並提出合理可靠的解決方案

比較常見的任務分解技術包括思路鏈(Chain of Thought) 和思路樹(Tree of Thoughts)，前者代表班路徑推理後者代表多路徑推理 (如果看得很矇回去好好看看 CoT、ToT是啥)

![image](https://hackmd.io/_uploads/H1BRqXQGke.png)


#### 有回饋規劃

先前提到的方法並未有回饋的設計，這在解決複雜的任務時面臨長期規劃的挑戰。為因應此挑戰，讓模型可以基於過去的行動和觀察、反思和調整執行計畫
其目的是糾正過去的錯誤並提升結果質量，其中試誤法是解決問題的關鍵步驟。兩種著名的反思或評估機制方法包括ReAct 和Reflexion。

* **ReAct** 它將推理與行動結合起來，透過在思考（Thought）、行動（Action）和觀察（Observation）這一系列步驟之間交替進行（重複N 次），幫助大語言模型解決複雜任務。

* **Reflexion** 則是透過參與者（Actor）、評估者（Evaluator）、自我反思（Self-Reflection）三者配合，透過參與者理解問題回答問題，再透過評估者評估該次的結果是否合理，最後送到自我反思去分析失敗原因，不足的點在哪並將結果存在長期記憶中，再次讓參與者產生下次結結果持續到結果合理。


### Memory

Agent 的記憶體模組負責記錄其內部日誌，涵蓋了先前的思考、行為及對環境的觀察，也包括與使用者的所有互動。在大語言模型 Agent 的研究文獻中，主要區分為兩種記憶體類型：

* 短期內存 - 涉及 Agent 當前狀態的上下文訊息，通常透過上下文內學習實現。由於上下文視窗的限制，這種記憶體是有時限和容量限制的。 (前幾次的 user prompt等)

* 長期記憶體 - 儲存 Agent 過去的行為和思考，這部分資訊需要在長時間內被保留並能夠被回憶起來。通常透過連接至快速、可擴展的外部向量儲存庫來實現，以便在需要時為 Agent 提供相關資訊。 (可能是以前得出的結果存在外部(SQL)或 ststem prompt 需要時調用)


### Tools

LLM 智能體透過工具與外界環境互動，這些工具包括Wikipedia 搜尋API、程式碼解釋器和數學引擎等。工具的範疇也涵蓋資料庫、知識庫和外部模型。智能體利用這些工具執行任務，透過工作流程取得資訊或完成子任務，以滿足使用者的需求。
(簡單來說就是你可能要怎麼獲取資料的方式，基本上自己寫 API 去獲取資料回來給 LLM 這樣)

一些常見的工具:

* **MRKL** 結合了LLM 和專家模組，後者可以是其他LLM 或特定功能的符號系統（如計算器或天氣API）。
* **Toolformer** 對LLM 進行微調，使其能夠使用外部工具API。
* **函數呼叫（Function Calling）** 功能增強了LLM 的工具使用能力，它定義了一套工具API 並將其作為請求的一部分提供給模型。
* **HuggingGPT** 利用LLM 作為任務規劃器，透過連接各種現有的AI 模型（根據描述）來解決AI 任務。


# RAG for LLMs

## 簡介
最簡單的定義: RAG 是一個將輸入與一組相關的支援文件結合起來的技術
(目的是讓目前沒有相關資訊的 LLM 可以獲取相關的文件資訊來回傳正確的結果)

![image](https://hackmd.io/_uploads/HkXal74zyx.png)

```
具體步驟如下：
輸入：是指LLM系統需要回答的問題。如果不使用RAG，問題直接由LLM回答。
索引：使用RAG時，會先將相關文件分塊，為這些區塊產生嵌入向量，並將它們索引到向量庫中。
     在進行查詢時，查詢內容也會以相似的方式進行嵌入。
檢索：透過比較查詢內容與索引向量，找到相關的文件。
生成：將找到的相關文件與原始提示結合作為額外上下文，然後傳遞給模型進行回應生成，
     最終形成系統對使用者的回答。
```

## RAG 演變

* 初級: 基本的 RAG Q+文件檢索
* 高級: 在文健檢索前會先優化資料，改善資料的細節度、優化索引結構等，檢索時透過改進嵌入模型來提高上下文的品質或使用更好的嵌入模型 (如OpenAI 的embeddings-ada-02 模型)，檢索後的優化過程我們專注解決情境窗限制和減少雜訊或會分散注意的資訊，以將更相關的內容放在提示前後，或重新計算查詢與文檔片段之間的語義相似度。此外，透過壓縮提示訊息也有助於解決這些問題。
* 模組化: 顧名思義，透過增強其功能模組來提升效能，例如加入相似性檢索的搜尋模組，以及在檢索工具上進行精細調整。模組化RAG 不僅包括了樸素RAG 和高階RAG 這兩種固定模式，還擴展了包括搜尋、記憶、融合、路由、預測和任務適配等多種模組，以解決各種問題

![image](https://hackmd.io/_uploads/ByTnQ7NM1g.png)


## RAG框架簡述

RAG 系統的核心組成部分，包括檢索(Retriever)、產生(Generator)和增強(Agumentation)三大環節的最新進展。這些組件共同構成了RAG 的基礎，使其能夠有效處理各種資訊檢索和內容生成的任務，進而提升系統的整體效能和應用彈性。


### Retriever 檢索

檢索的技術可以分成三類，提升語意理解、查詢與文件的精準匹配、檢索器與大語言模型的協同最佳化

* 提升語意理解

資料分塊策略：確定合適的資料分塊方式非常關鍵，這依賴於你的資料內容和應用需求。例如，針對單句效果更佳的句子轉換器，與處理256或512個詞元的文字時，使用文字嵌入-ada-002模型會更加出色。此外，還需考慮用戶提問的長度、應用需求和詞元數量上限。在實際應用中，透過嘗試不同的資料分塊策略來發現最優的檢索效率是常見做法。

專業領域的嵌入模型微調：確定了有效的資料分塊策略後，如果你的工作聚焦在特定領域，可能還需要對嵌入模型進行微調。否則，可能會導致系統無法正確理解使用者查詢。可針對廣泛的領域知識或特定的下游任務進行微調。例如，BGE-large-EN 開發的BAAI模型，就能透過微調來提高檢索的相關性。

* 查詢與文件的精準匹配

查詢重寫：透過多種技術改寫查詢，以提高配對的準確性，例如利用Query2Doc、ITER-RETGEN和HyDE等工具。

查詢嵌入的最佳化：透過調整查詢的嵌入表示，使其更好地與任務相關的潛在空間對齊，從而提升查詢效果。

* 檢索器與大語言模型的協同最佳化

優化檢索技術:透過分析大語言模型(LLM)提供的回饋，進一步完善檢索系統。例如，透過適應性增強檢索技術( AAR)，REPLUG，和UPRISE等方式來實現。

引入輔助工具:透過加入外部工具，如PRCA，RECOMP，和PKG，輔助優化資訊對齊過程。

### Generator 產生

檢索後處理與模型固定:在保持大語言模型(LLM)不變的情況下，透過後處理技術改善檢索結果的質量，如透過資訊簡化和結果優先排序等手段。資訊簡化有助於減少冗餘訊息，解決模型處理長文本的限制，並提升最終文本的生成品質。優先排序則是將最相關的資訊排在前面，以提高檢索的準確性。

針對RAG系統的LLM微調:為了提高RAG系統的效率，可以對生成文本的過程進行細緻調整或微調，確保生成的文本既自然流暢又能有效地結合檢索到的文檔信息。

### Agumentation 增強

增強階段： RETRO範例展示如何從零開始利用檢索增強進行大規模預訓練；它額外引入了一個基於外部知識建構的編碼器。此外，透過結合RAG 進行微調，可以進一步提升系統的效能。在推理階段，根據具體任務需求採取多種技術將檢索內容有效融入，以優化RAG 的應用效果。

增強資料來源：選擇何種增強資料對RAG 模型的效果影響極大。資料來源主要分為三類：非結構化資料、結構化資料、大語言模型產生的資料。

增強過程：對於一些需要多步驟推理的問題，單次檢索可能不足以解決，因此提出了以下幾種方法：

```
迭代檢索：模型透過多輪檢索，不斷深化、豐富資訊內容。        
         例如，RETRO和GAR-meets-RAG就是採用這種方法。
         
遞歸檢索：在這種方法中，一次檢索的輸出成為另一次檢索的輸入，逐步深入挖掘複雜查詢的相關信息，
         適用於學術研究和法律案例分析等場景。著名實踐包括IRCoT和Tree of Clarifications。
         
自適應檢索：根據特定需求調整檢索過程，選擇最適合的時機和內容進行檢索，以達到最佳效果。
         這種方法的代表性研究包括FLARE和Self-RAG。
```


# ❎ 剩下的之後再說再看，先去學 code


![image](https://hackmd.io/_uploads/r1rrCUVfJe.png)


=================================================================

