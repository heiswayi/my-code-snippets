Script to download NPM package (*.tgz) from GitLab Package Registry instead of using NPM registry (.npmrc).

```js
const https = require('https');
const fs = require('fs');
const path = require('path');

const DOWNLOAD_FOLDER = '.temp';
if (!fs.existsSync(DOWNLOAD_FOLDER)) {
  fs.mkdirSync(DOWNLOAD_FOLDER);
}

// Get the token from command line arguments
const token = process.argv[2];
if (!token) {
  console.error('Usage: node download-npm-packages.js <gitlab_token>');
  process.exit(1);
}

const packages = [
  {
    project_id: 123456,
    package_name: '@mycompany/libone',
    version: '1.0.1238989'
  },
  {
    project_id: 234567,
    package_name: '@mycompany/libtwo',
    version: '1.0.65457091'
  },
]

const options = {
  headers: {
    'Authorization': 'Bearer ' + token
  }
};

const downloadFile = (url, callback) => {
  const fileName = path.basename(url);
  const filePath = path.join(DOWNLOAD_FOLDER, path.join(__dirname, fileName).split('%2F')[1]);

  https.get(url, options, (res) => {
    if (res.statusCode !== 200) {
      console.error(`Failed to download ${decodeURIComponent(fileName)}: ${res.statusMessage}`);
      res.resume(); // Consume response data to free up memory
      return callback();
    }

    const fileStream = fs.createWriteStream(filePath);
    res.pipe(fileStream);

    fileStream.on('finish', () => {
      fileStream.close(() => {
        console.log(`Downloaded: ${decodeURIComponent(fileName)}`);
        callback();
      });
    });
  }).on('error', (err) => {
    console.error(`Error downloading ${decodeURIComponent(fileName)}: ${err.message}`);
    callback();
  });
};

const downloadAllFiles = (urls) => {
  let index = 0;
  const next = () => {
    if (index < urls.length) {
      downloadFile(urls[index], () => {
        index++;
        next();
      });
    }
  };
  next();
};

const encodePackages = (packages) => {
  return packages.map(pkg => {
    return `https://gitlab.com/api/v4/projects/${pkg.project_id}/packages/npm/${encodeURIComponent(pkg.package_name)}/-/${encodeURIComponent(pkg.package_name)}-${pkg.version}.tgz`;
  });
};

downloadAllFiles(encodePackages(packages));
```
