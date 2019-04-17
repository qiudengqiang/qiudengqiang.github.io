# 准备工作
homebrew
nvm
npm
# 初使化
npm install -g hexo-cli
cd <blog folder>
npm install

# 设置发布
git clone git@github.com:qiudengqiang/qiudengqiang.github.com.git public

# 设置皮肤
mkdir themes
cd themes
git clone git@github.com:qiudengqiang/jacman.git

# 操作Hexo
hexo clean  
hexo g
hexo s
hexo d

