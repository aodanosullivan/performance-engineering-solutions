#### New articles

This is a brief information for adding new articles.

##### Command line

 * create github account
 * create ssh key to be used for this service
   * run command `ssh-keygen -t ed25519 -C "github"`
   * accept defaults
   * when keys are created move them (id_ed25519 and id_ed25519.pub) to `~/.ssh/keys/github`
   * config private key to be used with github by editing `~/.ssh/config` and adding:

         Host github.com
          IdentityFile ~/.ssh/keys/github/id_ed25519
          IdentitiesOnly yes

 * add ssh key for ssh-like access
   * click profile icon on github
   * pick `Settings`
   * pick `SSH and GPG keys` on left menu
   * click `New SSH key`
   * add created (content of  ~/.ssh/keys/github/id_ed25519.pub) or other existing public key
 * ask Aodan for setting you as collabolator
 * clone with `git clone git@github.com:aodanosullivan/performance-engineering-solutions.git`
 * create branch for you changes
 * add 150x150 icon for your avatar or profile picture to `source/img/team` folder
 * update `source/team/index.md` with your resume and link to picture
 * add article data under `source/_posts` using other articles code as example
 * add header photo to be listed on summary page (refferenced in articles in header photos section) to folder `source/img/posts`, size should be same as existing examples
 * create short introduction, without md formatting in the beginning of the article, before <!--more--> tag, it will be listed on summary page before `Continue reading` section
 * commit changes and push to remote branch `git push origin your-branch`
 * after reviwe your changed will be merged and available to see on https://performance-engineering-solutions.com

##### Web browser

You can do similar thing by simply uploading files into mentioned locations with web browser, without using git from command line. You just need to be added as collabolator by owner - Aodan.

#### Testing content locally

You can check how your article looks like by setting up website locally

 * copy repo content with your changes to /tmp: `cp -r performance-engineering-solutions /tmp`
 * install hexo there: `cd /tmp/performance-engineering-solutions && npm install hexo`
 * generate code: `node_modules/hexo/bin/hexo generate`
 * run server: `node_modules/hexo/bin/hexo server`

Now you can see your changes tested
