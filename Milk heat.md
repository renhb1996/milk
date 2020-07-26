### Milk heat

```R
model_data <- wholemilk_alldata[,2:314]
model_data$Tt <- as.factor(model_data$Tt)
names(model_data) <- c(paste("w",names(wholemilk_alldata[,2:313]),sep=""),"Tt")
library(randomForest)# rf
library(e1071)# svm
library(lattice)
require(ggplot2)# plot
library(caret)# confusionMatrix
library(kknn)# knn
library(MASS)
library(sampling)# strata
library(ggplot2)

sub_train_all <- strata(model_data,stratanames=("Tt"),size=rep(34,16),method="srswor")
trainset_0 <- model_data[sub_train_all$ID_unit,]
testset <- model_data[-sub_train_all$ID_unit,]

sub_train_val <- strata(trainset_0,stratanames=("Tt"),size=rep(25,16),method="srswor")
trainset <- trainset_0[sub_train_val$ID_unit,]
valset <- trainset_0[-sub_train_val$ID_unit,]
# rf调参
par <- data.frame()
for (i in 1:500){
  for (j in 1:19){
    rf.model <- randomForest(Tt ~ ., data = model_data,importance = TRUE, proximity = FALSE,ntree = i, mtry = j)
    rf.pred <- predict(svm.model,testset[,-313], type =  "class", decision.values=TRUE, probability = TRUE)
    rf.table <- table(rf.pred, testset$Tt)
    conf <- confusionMatrix(rf.table)
    Precision_Recall_F1_Accuracy <- as.data.frame(conf$byClass)[,c(5:7,11)]
    #Precision_Recall_F1$Accuracy <- conf$overall[1]
    #Accuracy <- conf$overall[1]
    Precision_Recall_F1_Accuracy$ntree <- rep(i, 16)
    Precision_Recall_F1_Accuracy$mtry <- rep(j, 16)
    #Precision_Recall_F1$ID <- rep(i, 16)
    Precision_Recall_F1_Accuracy$Kinds <- level
    #rf_par <- rbind(rf_par, Precision_Recall_F1)
  }
  print(i)
}


###ntree=10, mtry=10000
err <- as.numeric()
for (i in 1:(length(names(trainset))-1)){
  mtry <- randomForest(Tt ~ .,data = trainset, mtry=i)
  err <- append(err,mean(mtry$err.rate))
  print(i)
}
print(err)
mtry_min <- which.min(err)
ntree_fit <- randomForest(Tt ~ .,data = trainset, mtry=mtry_min, ntree=10000)
plot(ntree_fit)
##



tuned <- tune.svm(Tt ~ .,data = model_data,
                 gamma = 10^(-6:1),
                 cost = 10^(-3:3))

level <- c('t0_0','t65_1','t65_10','t65_20','t65_3','t65_30','t75_10','t75_20','t75_3','t75_30','t85_10','t85_20','t85_3','t95_10','t95_15','t95_3')
svm_par <- data.frame()
for (i in 1:10) {
  set.seed(i)
  sub_train_all <- strata(model_data,stratanames=("Tt"),size=rep(32,16),method="srswor")
  trainset <- model_data[sub_train_all$ID_unit,]
  testset <- model_data[-sub_train_all$ID_unit,]
  
  svm.model <- svm(Tt ~ ., data = trainset,kernel = "radial",
                   gamma = 0.01, cost = 100,
                   probability = TRUE)
  svm.pred <- predict(svm.model,testset[,-313], type =  "class", decision.values=TRUE, probability = TRUE)
  svm.table <- table(svm.pred, testset$Tt)
  conf <- confusionMatrix(svm.table)
  Precision_Recall_F1 <- as.data.frame(conf$byClass)[,c(5:7,11)]
  #Precision_Recall_F1$Accuracy <- conf$overall[1]
  Precision_Recall_F1$ID <- rep(i, 16)
  Precision_Recall_F1$Kinds <- level
  svm_par <- rbind(svm_par, Precision_Recall_F1)
  print(i)
}


rf_par <- data.frame()
for (i in 1:10) {
  set.seed(i)
  sub_train_all <- strata(model_data,stratanames=("Tt"),size=rep(32,16),method="srswor")
  trainset <- model_data[sub_train_all$ID_unit,]
  testset <- model_data[-sub_train_all$ID_unit,]
  rf.model <- randomForest(Tt ~ ., data = model_data,importance = TRUE, proximity = FALSE,ntree = 500)
  rf.pred <- predict(svm.model,testset[,-313], type =  "class", decision.values=TRUE, probability = TRUE)
  rf.table <- table(rf.pred, testset$Tt)
  conf <- confusionMatrix(rf.table)
  Precision_Recall_F1 <- as.data.frame(conf$byClass)[,c(5:7,11)]
  #Precision_Recall_F1$Accuracy <- conf$overall[1]
  Precision_Recall_F1$ID <- rep(i, 16)
  Precision_Recall_F1$Kinds <- level
  rf_par <- rbind(rf_par, Precision_Recall_F1)
  print(i)
}

###调参
sub_train_all <- strata(model_data,stratanames=("Tt"),size=rep(32,16),method="srswor")
trainset <- model_data[sub_train_all$ID_unit,]
testset <- model_data[-sub_train_all$ID_unit,]
t <- c()
for (i in 1:10) {
  knn.model <- kknn(Tt ~., trainset, testset[,-313], kernel = "rectangular", k=i)
  conf <- confusionMatrix(testset$Tt,knn.model$fitted.values)
  #Precision_Recall_F1 <- as.data.frame(conf$byClass)[,c(5:7,11)]
  #Precision_Recall_F1$Accuracy <- conf$overall[1]
  t <- rbind(t, conf$overall[1])
  print(i)
}
k <- which.max(t)
print(k)
###
###
#取distance=2
milk.tkknn <- train.kknn(Tt~.,data = model_data,kernel = c("rectangular", "triangular", "epanechnikov", "optimal"),scale=T)
plot(milk.tkknn)
milk.tkknn$MISCLASS #显示错误率
milk.tkknn #输出最优参数情况


####
knn_par <- data.frame()
for (i in 1:10) {
  set.seed(i)
  sub_train_all <- strata(model_data,stratanames=("Tt"),size=rep(32,16),method="srswor")
  trainset <- model_data[sub_train_all$ID_unit,]
  testset <- model_data[-sub_train_all$ID_unit,]
  knn.model <- kknn(Tt ~., trainset, testset[,-313], kernel = "rectangular", k=3)
  conf <- confusionMatrix(testset$Tt,knn.model$fitted.values)
  Precision_Recall_F1 <- as.data.frame(conf$byClass)[,c(5:7,11)]
  #Precision_Recall_F1$Accuracy <- conf$overall[1]
  Precision_Recall_F1$ID <- rep(i, 16)
  Precision_Recall_F1$Kinds <- level
  knn_par <- rbind(knn_par, Precision_Recall_F1)
  print(i)
}

lda_par <- data.frame()
for (i in 1:10) {
  set.seed(i)
  sub_train_all <- strata(model_data,stratanames=("Tt"),size=rep(32,16),method="srswor")
  trainset <- model_data[sub_train_all$ID_unit,]
  testset <- model_data[-sub_train_all$ID_unit,]
  lda.model <- lda(Tt~., data = trainset)
  lda.pred <- predict(lda.model,testset[,-313], type =  "class", decision.values=TRUE, probability = TRUE)#predict(rf.model, testset[,-1057],type="prob")
  lda.table <- table(lda.pred$class, testset$Tt)
  conf <- confusionMatrix(lda.table)
  Precision_Recall_F1 <- as.data.frame(conf$byClass)[,c(5:7,11)]
  #Precision_Recall_F1$Accuracy <- conf$overall[1]
  Precision_Recall_F1$ID <- rep(i, 16)
  Precision_Recall_F1$Kinds <- level
  lda_par <- rbind(lda_par, Precision_Recall_F1)
  print(i)
}


models_par <- rbind(svm_par, rf_par, knn_par, lda_par)
models_par$models <- rep(c("SVM","RF","kNN","LDA"), each = nrow(svm_par))
models_par$models <- factor(models_par$models,order = TRUE,levels = c("RF","SVM","kNN","LDA"))
names(models_par)[4] <- "Accuracy"
library(ggpubr)
compare_means(Accuracy ~ models, data = models_par,
              method = "t.test")
# Visualize: Specify the comparisons you want
my_comparisons <- list( c("RF", "SVM"), c("RF", "kNN"), c("RF", "LDA"), c("SVM","kNN"),
                        c("SVM","LDA"), c("kNN","LDA"))

##Accuracy
ggboxplot(models_par, x = "models", y = "Accuracy",
          color = "models", palette = "jco")+ 
  stat_compare_means(comparisons = my_comparisons, method = "t.test",label = "p.signif")#+
##Precision
ggboxplot(models_par, x = "models", y = "Precision",
          color = "models", palette = "jco")+ 
  stat_compare_means(comparisons = my_comparisons, method = "t.test",label = "p.signif")#+

#Recall
ggboxplot(models_par, x = "models", y = "Recall",
          color = "models", palette = "jco")+ 
  stat_compare_means(comparisons = my_comparisons, method = "t.test",label = "p.signif")#+

#F1
ggboxplot(models_par, x = "models", y = "F1",
          color = "models", palette = "jco")+ 
  stat_compare_means(comparisons = my_comparisons, method = "t.test",label = "p.signif")#+





##project_2
models_par$Kinds <- as.factor(models_par$Kinds)# samples
models_par$models <- as.factor(models_par$models)# methods

#Accuracy
ggboxplot(models_par, x="Kinds", y="Accuracy", color = "models",
          palette = "jco", add = "jitter")+
  stat_compare_means(aes(group=models), label = "p.signif", label.y = 1.5,  method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))

#Precision
ggboxplot(models_par, x="Kinds", y="Precision", color = "models",
          palette = "jco", add = "jitter")+
  stat_compare_means(aes(group=models), label = "p.signif", label.y = 1.5,  method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))

#Recall
ggboxplot(models_par, x="Kinds", y="Recall", color = "models",
          palette = "jco", add = "jitter")+
  stat_compare_means(aes(group=models), label = "p.signif", label.y = 1.5,  method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))

#F1
ggboxplot(models_par, x="Kinds", y="F1", color = "models",
          palette = "jco", add = "jitter")+
  stat_compare_means(aes(group=models), label = "p.signif", label.y = 1.5,  method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))



##project_3
models_par$Kinds <- as.factor(models_par$Kinds)# samples
models_par$models <- as.factor(models_par$models)# methods
models_par_65 <- subset(models_par, Kinds == c("t0_0","t65_1","t65_10","t65_20","t65_3","t65_30"))
models_par_65$Kinds <- factor(models_par_65$Kinds,order = TRUE,levels = c("t0_0","t65_1","t65_3","t65_10","t65_20","t65_30"))

#Accuracy
ggboxplot(models_par_65, x="Kinds", y="Accuracy", color = "models",
          palette = "jco")+
  stat_compare_means(aes(group=models), label = "p.signif", label.y = 1.5,  method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))

#Precision
ggboxplot(models_par_65, x="Kinds", y="Precision", color = "models",
          palette = "jco")+
  stat_compare_means(aes(group=models), label = "p.signif", label.y = 1.5,  method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))

#Recall
ggboxplot(models_par_65, x="Kinds", y="Recall", color = "models",
          palette = "jco")+
  stat_compare_means(aes(group=models), label = "p.signif", label.y = 1.5,  method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))

#F1
ggboxplot(models_par_65, x="Kinds", y="F1", color = "models",
          palette = "jco")+
  stat_compare_means(aes(group=models), label = "p.signif", label.y = 1.5,  method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))


##
##
##75
models_par_75 <- subset(models_par, Kinds == c("t0_0","t75_10","t75_20","t75_3","t75_30"))
models_par_75$Kinds <- factor(models_par_75$Kinds,order = TRUE,levels = c("t0_0","t75_3","t75_10","t75_20","t75_30"))
#Accuracy
ggboxplot(models_par_75, x="Kinds", y="Accuracy", color = "models",
          palette = "jco")+
  stat_compare_means(aes(group=models), label = "p.signif", label.y = 1.5,  method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))

#Precision
ggboxplot(models_par_75, x="Kinds", y="Precision", color = "models",
          palette = "jco")+
  stat_compare_means(aes(group=models), label = "p.signif", label.y = 1.5,  method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))

#Recall
ggboxplot(models_par_75, x="Kinds", y="Recall", color = "models",
          palette = "jco")+
  stat_compare_means(aes(group=models), label = "p.signif", label.y = 1.5,  method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))

#F1
ggboxplot(models_par_75, x="Kinds", y="F1", color = "models",
          palette = "jco")+
  stat_compare_means(aes(group=models),   method = "anova")+
  theme(axis.text.x = element_text(angle = 45, hjust = 0.5, vjust = 0.5))

```

