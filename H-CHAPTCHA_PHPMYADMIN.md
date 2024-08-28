<h1>H Captcha</h1>
<pre><code>
  $cfg['CaptchaApi'] = 'https://www.hcaptcha.com/1/api.js';
  $cfg['CaptchaCsp'] = 'https://hcaptcha.com https://*.hcaptcha.com';
  $cfg['CaptchaRequestParam'] = 'h-captcha';
  $cfg['CaptchaResponseParam'] = 'h-captcha-response';
  $cfg['CaptchaSiteVerifyURL'] = 'https://hcaptcha.com/siteverify';
  // This is the secret key from hCaptcha dashboard
  $cfg['CaptchaLoginPrivateKey'] = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx';
  // This is the site key from hCaptcha dashboard
  $cfg['CaptchaLoginPublicKey'] = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx';
</code></pre>
<br/>
<pre><code>https://docs.phpmyadmin.net/en/latest/config.html?highlight=recaptcha#recaptcha-using-hcaptcha</code></pre>
<br/>
<pre><code>https://dashboard.hcaptcha.com/</code></pre>
