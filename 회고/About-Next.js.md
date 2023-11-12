# Next.js
## Next.js의 환경 설정
Next.js는 아주 좋은 프레임워크라는 느낌이 든다. 하지만 너무나도 예리한 칼날과 같아서, 정확하게 사용할 줄 모르면 정말 어렵다.  
그리고, 정확하게 사용하기까지 필요한 시간이 꽤 오래 걸릴 것 같다.   
Python의 DJango Framework는 오마카세 또는 부페의 느낌이라면, Next.js는 아주 잘 갖춰진 Modern self bar 같았다.  
하지만, Vercel의 Next.js 프레임워크는 React의 mainstream이 되었고, 앞으로도 쭉 mainstream으로 남을 것 같다.  
### Issue
[[개발] Sever side에서 Firestore 연결하여 데이터 fetch&store 후 Client side로 넘기는 API Endpoint 생성](https://github.com/Yoonkeee/useRemind_dev_/issues/1)  

환경 변수를 불러오는 것이 아주 간단한 것처럼 보이지만, 이를 client side에 넘기는 것은 정말 다른 차원의 일이었다.  
next.js는 어찌나 철두철미한지, NEXT_PUBLIC_의 PREFIX가 붙지 않은 환경 변수의 사용은 그 어떠한 방법으로도 client side로 넘길 수 없었다.  
하지만, 환경 변수에 NEXT_PUBLIC_을 사용하면 이는 번들에 값이 line-up되므로, 사용해선 안된다. 즉, framework가 권고하는 방법이지만 역설적으로 이를 사용할 수 없었다.  
어떻게든 찾아낼 수 있겠지만, 한정된 시간과 체력으로 인해 server side에서 필요한 일을 다 하기로 했다.  
덕분에 이 과정에서 Next.js가 제공하는 라우팅 시스템이 얼마나 편리하고 쉬운지도 느낄 수 있었다.

eof
