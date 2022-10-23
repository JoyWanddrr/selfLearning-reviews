# 2-3學期的自學經驗回顧

## 所遇到的問題
    寫作業的時候，程式碼反覆鬼打牆。
  
  ### 具體描述
      以實作Url Shortener為例，依據條件式的不同，理應渲染不同的頁面或者效果。但卻只會實際出現第一項判斷式的答案。不管接收到甚麼都一樣(inputUrl=req.body.url)。

      舉例(這是最後成功的版本):

      `
      router.post('/', (req, res) => {
        const hostUrl = req.headers.host
        const inputUrl = req.body.url
        Url.findOne({inputUrl })
          .lean()
          .then((urlData) => {
            if (urlData) {
            res.render('error', { shortUrl: urlData.shortUrl, inputUrl, hostUrl })
            }
            else {
              Url.create({ originalUrl: inputUrl, shortUrl: shortener(5) })
                .then((urlData) =>
                  res.render('index', { shortUrl: urlData.shortUrl, inputUrl, hostUrl }))
            }
          })
          .catch(err => console.log(err))
        })
      `

      這段程式碼我大概實驗了快3個小時。沒成功的版本，沒有任何錯誤訊息。為了找出錯誤在哪裡，我把判斷式一行一行分開寫。

      `
      if(urlData===null) //最一開始，資料庫還沒有任何東西的時候。
      if(urlData===urlData) //資料庫是否有相符的資料。
      if(urlData!==urlData) //資料庫是否沒有相符的資料。
      `

      只要超過一種判斷以上，我的程式碼就只會印出他執行過的第一行。
      具體表現是，假設判定urlData為true，渲染error頁面。但之後就算是新輸入的資料，也只會出現error頁面，反之亦然。

  ### 問題
      最終發現是我的Url.findOne沒有把({ originalUrl: })給加上去所導致。
      原因推測造成他永遠只判斷inputUrl，卻沒有比對資料庫。

  ### 其他問題
      中間還遇到一些，因為我太想把所有東西都寫在一起。所以出現類似下面這種程式碼。

      `
      router.post('/renew', (req, res) => {
        const hostUrl = req.headers.host
        const inputUrl = req.body.url
        Url.findOne({inputUrl })
          .lean()
          .then((urlData) => {
            if (urlData) {
            res.render('error', { shortUrl: urlData.shortUrl, inputUrl, hostUrl })
            }
            else {
              Url.create({ originalUrl: inputUrl, shortUrl: shortener(5) })
                .then((urlData) =>
                  res.render('renew', { shortUrl: urlData.shortUrl, inputUrl, hostUrl }))
            }
          })
          .then((urlData)=>res.render('/')) //程式碼B
          .catch(err => console.log(err))
        })
      `

      程式碼B會與上面同樣有做render的程式碼衝突，這個他就會報錯。但覺得犯這種錯誤的自己有點好笑，是有多不想寫新的。
