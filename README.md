# Painting_classifier

# Painting_classifier

Overall model test accuracy: 51% 
![image](https://user-images.githubusercontent.com/96949018/233967330-784474ce-5bd1-490d-ba42-b25257c429b3.png)

	資料集下載
  將資料及下載下來後上傳到自己的google雲端硬碟中，並且開啟分享，透過openid存取，避免浪費學術網路資源 
  ![image](https://user-images.githubusercontent.com/96949018/233967378-4e00e01f-65e3-42f1-9e18-ae34014d616b.png)

	取得資料集:
	make_author_dic(): 從artists.csv中取出作者名稱，並將名稱中間的空格改以’_’連接起來。
	class_weight: 因為每個畫家之間的畫作數量很不平均，會造成 class 資料不平衡的問題，並影響到模型的訓練。透過計算權重 (class_weight) 以便後續訓練模型時使用 (資料越多權重越低)，每個畫家的權重為所有畫的數量總合除上每個畫家的畫的數量乘畫家數
class_weight[i]=所有畫的數量/(畫家數 * 畫家i的畫的數量)
![image](https://user-images.githubusercontent.com/96949018/233967465-244546e2-873c-42b5-838e-d5fead99301d.png)

 
	資料前處理
	get_label(pic_name): 將圖片的檔名轉換成先前建立好的class_name字典中對應的label。 
  ![image](https://user-images.githubusercontent.com/96949018/233967512-ac033314-47bf-4c75-9eaf-4aa7de3d344e.png)

	get_path(dir, img_name): 使用os.path.join()，將dir與img_name的字串成為一個完整路徑連接起來。 
  ![image](https://user-images.githubusercontent.com/96949018/233967542-be7df752-538d-400e-8599-d77e1ef2e297.png)

	make_path_label(dir): 將dir中的所有圖片轉換成路徑與label，供後續圖片前處理做使用，透過tensorflow.keras.utils 中的to_categorical套件，將label轉換成onehot encoding形式的陣列。 
  ![image](https://user-images.githubusercontent.com/96949018/233967597-e64af271-98bf-40bf-86b9-878b74af4ae3.png)

	get_img(path): 將圖片的大小設定為(256, 256)，因為若設為512，系蓊RAM會不足，無法訓練；並且將圖片每個 pixel 映射到 [0,1] 之間，透過將每個pixel除上最大值255 
  ![image](https://user-images.githubusercontent.com/96949018/233967644-d2518d36-6232-46bc-93e1-986c466b8491.png)

	make_dataset(dir): 將圖片讀入並做前處理，後在打包成tensorflow dataset形式，SHFFLE_BUFFER為每次隨機選取圖片時的圖片池大小，若設太大，系統RAM會不夠。 
	將訓練資料即拆分成80%訓練，20%驗證，已在訓練時實時觀測訓練情況；BATCH_SIZE設為32，為了不讓訓練速度太快。
  ![image](https://user-images.githubusercontent.com/96949018/233967704-a3b3a83a-544c-4006-b8b2-605d545a05e2.png)


	建立模型
	使用Keras Sequential API建立模型，加入Conv2D layer，激活函數為ReLU(Rectangle Linear Unit)，能夠有效抓取圖片的特徵地圖，MaxPooling2D layer將顯著特徵值保留，並且減少計算量，BatchNormalization layer將輸入標準化。
	Conv2D layer的filter數值在此模型中宜為由小到大(64, 128, 256, 512)，依此順序設定模型的表現較好。
	後再將所有特徵地圖做GlobalAveragePooling2D，將多維特徵拉為一維向量，GlobalAveragePooling2D具有許多優點，例如減少需要訓練的參數數量、防止過度擬合、提高網絡的generalization等。此外，它還可以提高網絡的計算速度。
	最後再將model的參數透過Dropout layer隨機捨棄參數，防止過度擬合。
	輸出層的Dense layer的激活函數為softmax，將機率最大者輸出。
	制定訓練計畫
	Optimezer設定為Adam，並將學習率設為定值0.0001
	損失函數設為categorical crossentropy，因輸出為長度50的機率向量
	Callbacks設為early stopping，透過監測validation data的loss，若validation loss在50個epoch內皆未下降，則將模型儲存為最佳狀態時的模型。
	Epoch設為100
	透過keras.models的load_model package，將模型儲存為’painting_classification.h5’，方便之後調用
	評估模型
	Model accuracy:  
  ![image](https://user-images.githubusercontent.com/96949018/233967827-f13b5d07-bb31-4616-8d45-0c6cef2ea3e4.png)

	Model loss:  
  ![image](https://user-images.githubusercontent.com/96949018/233967839-f85f64bd-1483-4262-9e9a-2a3c2e00097d.png)

	做預測
	透過numpy 的expand_dims function將圖片轉換成4維數據(1, 256, 256, 3)
	model.predict(img)產生長度為50的結果
	透過np.argmax()產生最大結果的畫家label
	使用rev_class_name字典得知名字
  ![image](https://user-images.githubusercontent.com/96949018/233967863-1b296be1-a36d-4202-8945-d7ffb0fcdf1c.png)

 
四、心得
透過這次的midterm project，完整的撰寫出資料前處理、模型建置與評估模型等等步驟，也嘗試用許多方法設法讓準確度升高，包括
	使用資料擴展技術(此方法註解在程式碼中): 將圖片透過旋轉或縮小，讓圖片集擴展，但此方法會導致過度擬合資料集，導致準確度不會超過30%，故此方法在在此模型中不適用。
	嘗試使用ResNet，手動建立類ResNet模型: 但我使用此方法準確率依然不高，可能是我使用方法有誤，我會再持續研究。
	嘗試各種超參數組合: 每改一次超參數組合，皆需要等待訓練許久，導致我colab gpu資源一直不夠，只能多開2隻帳號繼續訓練；程式碼中的超參數及模型是經過各種排列組合後得出的結果，其中發現有幾項此模型的特徵: 
	Conv2D 的filter數量需逐漸升高
	每次Conv2D後需要一層MaxPooling2D與BatchNormalization來取出特徵
	將資料拉為一維時，GlobalAveragePooling2D較Flatten 表現好
	Dropout參數超過0.5會導致模型訓練準確度很低
總結來說，要使用深度學習來預測名畫的作者準確率很難提高，因為畫作特徵極其難抓取，可能需要引入pretrained model，才能提高準確度。
