I have a small application via REST API, which only need some item_id to verify the user insead of full user authentication

1. Changed the file "vendor/yiisoft/yii2/filters/auth/CompositeAuth.php"
```
    public function authenticate($user, $request, $response)
    {
		  ......
      ......
            if ($identity !== null) {
                return $identity;
            }
        }		
		
    //our codes to find out if there is an access_token 
		$bearerToken = \Yii::$app->getRequest()->getQueryParam('access_token');
		if ($bearerToken != ""){
			return true;
		}
		//end of our codes
		
        return null;
    }
```


2. Changed the file "vendor/filsh/yii2-oauth2-server/filters/ErrorToExceptionFilter.php"

```
    public function afterAction($event)
    {
        $response = Yii::$app->getModule('oauth2')->getServer()->getResponse();
		//\Yii::info($response, 'oauth'); 
        $isValid = true;
        if($response !== null) {
            $isValid = $response->isInformational() || $response->isSuccessful() || $response->isRedirection();
        }

		//our codes: verifying the access_token
		$bearerToken = \Yii::$app->getRequest()->getQueryParam('access_token');
		if ($bearerToken != null){
			$isValid = $this->isAccessTokenValid($bearerToken);
        }
		//end of our codes
		
		
        if(!$isValid) {
            throw new HttpException($response->getStatusCode(), $this->getErrorMessage($response), $response->getParameter('error_uri'));
        }
    }
```
New Function Created:

```
public static function isAccessTokenValid($token)
    {
        if (empty($token)) {
            return false;
        }
        $query = (new \yii\db\Query())
            ->select(['access_token','expires'])
            ->from('oauth_access_tokens')
            ->where(['access_token' => $token])
//            ->limit(10)
            ->one();
        if (empty($query)) {
            return false;
        }
        $expire = $query['expires'];
        return $expire > date('Y-m-d h:i:s');
    } 
```

3. ItemID controller added the codes

```
	public function findByDeviceId($item_id)
	{
				
		$count = ItemId::find()->where(['item_id' => $item_id])->count();
		
		if ($count == 0 ) {
			return false;
		}
		return true;
	}

```
