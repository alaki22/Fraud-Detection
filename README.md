# IEEE-CIS Fraud Detection
მოკლე მიმოხილვა: გვაქვს დიდი დატასეტი,ტრანზაქციების და იდენტობების ცხრილი და ამ ინფორმაციის მიხედვით უნდა შევისწავლოთ ტრანზაქცია არის თუ არა Fraud.
ჩემი მიდგომა: რადგანაც მოცემული დატასეტი დიდი იყო, დავჯავი 3 ნაწილად:train, validation და test 70/15/15 პროპორციით. შემდეგ ვქენი Feature Engineering, Feature Selection და ამ გასუფთავებულ და შემცირებულ დატასეტზე შემდეგ დავატრენინგე სხვადასხვა მოდელი და შევაფასე ROC_AUC-ის მიხედვით.

# Repository Structure
```
Fraud-Detection/
│
├── README.md
├── model-experiment-adaboost.ipynb
├── model-experiment-desicion-tree.ipynb
├── model-experiment-logistic-regression.ipynb
├── model-experiment-randomforest.ipynb
├──model-experiment-xgboost.ipynb
└── model-inference.ipynb
```

# Feature Engineering
Nan მნიშვნელობების დამუშავება: შევქმენი კლასი Nan მნიშვნელობების შესავსებად. ამ დატასეტში იყო ისეთი სვეტები, რომლებიც დიდი პროცენტულობით შეიცავდა Nan-ს. Threshold-ად ავიღე შედარებით მაღალი რიცხვი: 0.95, რომ ძალიან ბევრი სვეტი არ გადამეყარა და მნიშვნელოვანი ინფორმაცია არ დამკარგვოდა. Numerical სვეტებში შესავსებად გამოვიყენე მედიანა, ხოლო categorical სვეტებში გამოვიყენე მოდა. 

კატეგორიული ცვლადების რიცხვითში გადაყვანა: კატეგორიულების რიცხვითში გადასაყვანადაც ცალკე კლასი შევქმენი.  გამოვიყენე One-Hot Encoding იმ კატეგორიული ცვლადებისთვის, რომელთა კატეგორიების რაოდენობა <= 3. გამოვიყენე Woe Encoding იმ კატეგორიული ცვლადებისთვის, რომელთა კატეგორიების რაოდენობა > 3. woe-ს გამოყენება შესაფერისად ჩავთვალე, რადგან კლასიფიკაციასთან გვქონდა საქმე და ეს Woe-ს "ძლიერი მხარეა".

# Feature Selection
ცვლადების შესარჩევად თავდაპირველად დავიწყე RFE-ის გამოყენება. თუმცა დიდ დატასეტთან გვქონდა საქმე, ხოლო RFE ყოველ ჯერზე ახლიდან აფასებს ყველა ცვლადს და ერთ იტერაციაზე ერთ ცვლადს აცილებს. ამის გამო ძალიან დიდი დრო მიჰქონდა.
ამიტომ შევცვალე სტრატეგია და გამოვიყენე კორელაცია,რომელიც პოულობს ერთნაირი კორელაციის მქონდე ცვლადებს და მათგან ერთ-ერთს აგდებს. default threshold-ად ავიღე 0,8.  ამ მიდგომამ ბევრად სწრაფი და უკეთესი შედეგი მომცა ვიდრე RFE-მ. 

# Training
1. Logistic Regression - ეს მოდელი გამოვიყენე რადგან საქმე გვქონდა Binary კლასიფიკაციასთან. რადგანაც გვქონდა არადაბალანსებული დატა, დაგავწყვიტე Undersampling და Oversampling-ის გამოყენება Logistic Regression-თან ერთად, თუმცა შედარებით დაბალი შედეგები მივიღე ამ შემთხვევებში,რასაც არ მოველოდი. აქედან დასკვნა გამოვიტანე რომ როგორც ჩანს Undersampling-ის დროს დაიკარგა მნიშვნელოვანი ინფორმაცია და მოდელმა ვერ შეძლო ამი გამო კარფად დასწავლა, ხოლო Oversampling-ის დროს კი ბევრმა ერთნაირმა sample-მა არასწორად დაატრეინეინებინა მოდელი. საბოლოოდ ყველაზე კარგი შედეგი ჩვეულებრივმა Logistic Regression-მა მომცა. მის ერთერთ პარამეტრად ავიღე class_weight='balanced'. ეს პარამეტრი მაღალ წონას ანიჭებს უმცირესობაში მყოფ კლასს და ამგვარად ცდილობს ჩენი დატასეტის დაბალანსებას. ასევე სანამ დავატრეინებდი, დატა დავა-scale-ე standardscaler-ის დახმარებით.
   roc_auc_train - 0.8595794563859439
   roc_auc_val - 0.8567290245815232
   roc_auc_test - 0.8563446277026276
2. Desicion Tree - ამ მოდელმაც დაახლოებით იგივე შედეგი მომცა რაც Logistic Regression-მა. გამოვიყენე პარამეტრები max_depth, min_samples_split,min_samples_leaf, class_weight, criterion. ამ პარამეტრებს ვიყენებ რომ ხე არ მოხდეს Overfit და იქამდე მოხდეს მისი Pruning. ხოლო class_weight-ს იგივე დანიშნულება აქვს, რაც Logistic Regression-შ ჰქონდა. შედეგის ცოტა გასაუმჯობესებლად ისევ გამოვიყენე Undersampling-oversampling და undersampling-ა მციორეოდენით უკეთესი შედეგი დადო.
   roc_auc_train - 0.8648983701276667
   roc_auc_val - 0.8483172859226424
   roc_auc_test - 0.8543011427229314.
3. RandomForest - ამ მოდელსაც ვიყენებ იგივე პარამეტრებით, რა პარამეტრებითაც ვიყენებ Desicion Tree-ს n_estimators-ის დამატებით, რომელიც რენდომ დაგენერირებული ხეების რაოდენობის განმსაზღვრელია. ამ მოდელმა შედარებით უკეთესი შედეგი მომცა რაც არ იყო გასაკვირი.
   roc_auc_train - 0.8871422521926219
   roc_auc_val - 0.8793878102186004
   roc_auc_test - 0.8817038855279284
4. AdaBoost - AdaBoost როგორც წესი ზემოთჩამოთვლილებზე უკეთესი მოდელია, მაგრამ სავარაუდოდ არადაბალანსებულობსი გამო ყველაზე ცუდი შედეგი ამან დამიდო.ვერც oversampling  დაეხმარა. ვიყენებ პარამეტრებს n_estimators და learning_rate.
   roc_auc_train - 0.8324381401019687
   roc_auc_val - 0.8357868499391151
   roc_auc_test - 0.8346919941223281
5. XGBoost - ეს მოდელი აღმოჩნდა საბოლოოდ საუკეთესო, არამარტო ROC_AUC, f1-იც ყველაზე მაღალი ჰქონდა. ამ მოდელის პარამეტრებზე სხვადასხვა ვარიანტი გავტესტე და საბოლოოდ ყველა კარგი შედეგი დადო მოდელმა რომლის პარამეტრებია: n_estimators=100, max_depth=10, learning_rate=0.1, min_child_weight=20, scale_pos_weight=2.33. scale_pos_weight - ეს პარამეტრი გამოიყენება არასდტაბილურობის დასაბალანსებლად. სხვები კი უმეტესად იმაზე ზრუნავენ Overfit არ მოხდეს.
  roc_auc_train - 0.959133545741893
  roc_auc_val - 0.9381028494801585
  roc_auc_test - 0.9383287621446105
   
# MLFLOW
Linear Regression - https://dagshub.com/alaki22/Fraud-Detection.mlflow/#/experiments/2/runs/8d48939068b2490db67eb36778785fc9

Desicion Tree - https://dagshub.com/alaki22/Fraud-Detection.mlflow/#/experiments/3/runs/afefe8bdfedd4d93a8152777b63956ae

RandomForest - https://dagshub.com/alaki22/Fraud-Detection.mlflow/#/experiments/4/runs/63eaacb4a1ff44f0995e7025cd27cadb

AdaBoost - https://dagshub.com/alaki22/Fraud-Detection.mlflow/#/experiments/5/runs/0500911a7fa04ad8be95fec596ea228e

XGBoost(second best) - https://dagshub.com/alaki22/Fraud-Detection.mlflow/#/experiments/6/runs/aa7610dd43764f0cacc048f66949f737

XGBoost - https://dagshub.com/alaki22/Fraud-Detection.mlflow/#/experiments/6/runs/c2c667334573419ea62e0fde98753a5b

