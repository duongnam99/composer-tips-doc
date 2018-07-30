#  [Martin Hujer blog](/)

## 17 lời khuyên để  sử dụng composer hiệu quả    

2018-01-05

Hầu hết lập trình viên PHP biết cách sử dụng Composer, thế nhưng không phải tất cả chúng ta đều đang sử dụng chúng một cách hiệu quả hoặc theo những cách tốt nhất có thể. Vì vậy tôi quyết định tổng hợp những thứ mà tôi nghĩ quan trọng đối với quy trình làm việc hàng ngày của tôi.  

Tử tưởng của hầu hết các lời khuyên là "thực hiện nó một cách an toàn", có nghĩa là nếu có nhiều cách để xử lí một vấn đề nào đó, tôi sẽ dùng cách tiếp cận với ít khả năng lỗi nhất. 

## Lời khuyên #1: Đọc tài liệu  

Tôi thực sự có ý đó. [Tài liệu](https://getcomposer.org/doc/) rất tuyệt và việc bỏ ra vài giờ đồng hồ ngồi đọc nó sẽ giúp bạn tiết kiệm thời gian về lâu dài. Bạn sẽ ngạc nhiên vì số thứ mà Composer có thể làm.  

## Lời khuyên #2: Hiểu rõ sự khác nhau giữa "project" và "library"  

Điều này rất quan trọng, rặng liệu bạn đang tạo một "project" hay một "library". Mỗi loại sẽ yêu cầu sự luyện tập riêng biệt.  

"library" là một package có thể tái sử dụng và bạn có thể thêm vào như một dependency - như là `symfony/symfony`, `doctrine/orm` hoặc [`elasticsearch/elasticsearch`](https://github.com/elastic/elasticsearch-php).  

1 project thường sẽ là 1 ứng dụng và phụ thuộc vào 1 vài thư viện. Nó thường không được tái sử dụng (những project khác không yêu cầu nó như là 1 dependency). Ví dụ điển hình là trang website thương mại, hệ thống hỗ trợ khách hàng, vân vân.  

Tôi sẽ phân biệt giữa "library" và project trong các lời khuyên bên dưới  


## Lời khuyên #3: Dùng các phiên bản dependency cụ thể  cho ứng dụng  

Neus bạn đang tạo một ứng dụng, bạn nên dùng phiên bản cụ thể nhất để miêu tả dependency.Nếu bạn cần phân tích file YAML, bạn nên chỉ rõ dependency như thế này: `"symfony/yaml": "4.0.2"`.

Thậm chí nếu thư viện tuân theo [Semantic Versioning](https://semver.org/), Vẫn có thể có sự phá vỡ khả năng tương thích ngược  giữa các bản nhỏ và bản vá. Lấy ví dụ, nếu bạn đang dùng `"symfony/symfony": "^3.1"`,Có thể những thứ không dùng trong 3.2 có thẻ phá honggr bài test ứng dụng của bạn, hoặc cũng có thể có các bug đã được sửa trong PHP_CodeSniffer và nó sẽ phát hiện các dạng vấn đề mới trong code của bạn, và 1 lần nữa nó có thể dẫn tới cấu trúc code hỏng.

Việc cập nhật các dependency cũng phải thật thận trọng, không thể tùy tiện được. 1 trong những lời khuyên bên dưới sẽ thảo luận về điều này chi tiết hơn.

Nghe có vẻ hơi quá, nhưng điều này sẽ ngăn việc các đồng nghiệp của bạn đột ngột cập nhật tất cả các dependency khi thêm 1 thư viện mới vào project (mà bạn có thể bỏ lỡ trong Code Review).  

## Lời khuyên #4: Sử dụng các khoảng phiên bản cho các dependency của thư viện  

Nếu bạn đang tạo một thư viện, bạn nên định nghĩa khoảng phiên bản rộng nhát có thể .Nếu bạn tạo một thư viện dùng `symfony/yaml` để phân tích cú pháp YAML, bạ nên require như sau:  
    
    
    "symfony/yaml": "^3.0 || ^4.0"
    

Có nghĩ là thư viện của bạn có thể sử dụng `symfony/yaml` với Symfony 3.x hoặc 4.x . It is important, điều này rất quan trọng vì những hạn chế này sẽ được chuyển tới ứng dụng sử dụng thư viện của bạn.  

Trong trường hợp có 2 thư viện với yêu cầu mâu thuẫn nhau, ví dụ 1 cái yêu cầu ~3.1.0 và cái kia yêu cầu ~3.2.0, thì quá trình cài đặt sẽ thất bại.  

## Lời khuyên #5: Bạn nên commit composer.lock lên git trong các ứng dụng  

Nếu bạn đang tạo 1 project, bạn chắc chắn muốn commit `composer.lock`lên git. Điều này sẽ đảm bảo tất cả mọi người - bạn, đồng nghiệp của bạn, server CI của bạn và server product của bạn - đang chạy ứng dụng với cùng các phiên bản dependency.

Thoạt nhìn, nó nghe có vẻ vô dụng - bạn đã sử dụng 1 phiên bản cụ thể theo những ràng buộc được đề cập trong lời khuyên #3. Nhưng không, cũng có những phụ thuộc trong những dependency của bạn lại không bị hạn chế bởi những ràng buộc này (ví dụ `symfony/console` phụ thuộc vào `symfony/polyfill-mbstring`). Vì vậy nếu không commit file composer.lock, bạn sẽ không thể lấy được những bộ dependency giống nhau.  

## Lời khuyên #6: Để  composer.lock trong .gitignore ở các thư viện.  
Nếu bạn đang tạo 1 thư viện (hãy gọi nó là `acme/my-library`), bạn không nên commit file composer.lock. Nó [Không tạo ảnh hưởng gì](https://getcomposer.org/doc/02-libraries.md#lock-file) cho các project đang sử dụng thư viện của bạn.  

Tưởng tượng rằng `acme/my-library` sử dụng `monolog/monolog` như là một dependency. Nếu bạn commit `composer.lock`, những người phát triển `acme/my-library` có thể đang sử dụng một phiên bản cũ của Monolog. Nhưng khi thư viện hoàn thành, và bạn sử dụng nó trong các project thật, 1 phiên bản mới của Monolog có thể được cài đặt, và nó không còn tương thích với thư viện nữa. Nhưng bạn lại không hề đề ý đến điều đó, bởi vì `composer.lock`.  

Vậy tốt nhất là để `composer.lock` vào trong `.gitignore` để bạn không vô tình commit nó lên.   

Nếu bạn muốn chắc chắn rằng thư viện tương thích với các phiên bản khác nhau của các dependency, đọc lời khuyên tiếp theo.  

## Tip #7: Chạy các kiến trúc Travis CI với nhiều phiên bản dependency  

> Lời khuyên này chỉ áp dụng cho các thư viện ( vì bạn sử dụng các phiên bản cụ thể cho các ứng dụng).  

Nếu bạn đang xây dựng 1 thư viện mã nguồn mở, bạn chắc hẳn đang sử dụng Travis CI để dựng kiến trúc của nó.  

Mặc định, Composer sẽ cài đặt phiên bản lớn nhất có thể của các dependency được cho phép bởi sự ràng buộc trong `composer.json`. Điều này có nghĩa là đối với dependency ràng buộc  `^3.0 || ^4.0`, các kiến trúc sẽ luôn sử dụng phiên bản lớn nhất của các v4 được phát hành. Vì phiên bản 3.0 không bao giờ được kiểm tra, thư viện có thể sẽ không tương thích với nó và điều này có thể khiến người dùng của bạn cảm thấy không buồn.  

May mắn là, Composer cung cấp 1 bộ chuyển đổi để cài đặt phiên bản thấp nhất có thể của dependency `--prefer-lowest` ( nên sử dụng `--prefer-stable` để tránh cài đặt các phiên bản không ổn định).  

Cập nhật cấu hình .travis.yml có thể trông như thế này:  

    
    
    language: php
    
    php:
      - 7.1
      - 7.2
    
    env:
      matrix:
        - PREFER_LOWEST="--prefer-lowest --prefer-stable"
        - PREFER_LOWEST=""
    
    before_script:
      - composer update $PREFER_LOWEST
    
    script:
      - composer ci
    

Xem trực tiếp [my mhujer/fio-api-php library](https://github.com/mhujer/fio-api-php/blob/master/.travis.yml) và [the build matrix on Travis CI](https://travis-ci.org/mhujer/fio-api-php)  

Mặc dù giải pháp này sẽ khắc phục được hầu hết các vấn đề không tương thích, hãy nhớ rằng có nhiều sự kết hợp của các dependency giữa những phiên bản thấp nhất và cao nhất. Và chúng có cũng có thể trở nên không tương thích với nhau. 

## Lời khuyên #8: Sắp xếp các package trong require và require-dev theo tên  

Sẽ rất hay nếu để các package trong mục `require` và `require-dev` được sắp sếp bởi tên. Nó có thể giúp tránh những xung đột gộp không cần thiết khi rebase một nhánh. Bởi vì nếu bạn thêm 1 package vào cuối danh sách trong 2 nhánh, sẽ luôn có những xung đột khi gộp.  

Đây là việc tẻ nhạt  khi làm thử công, nên tốt nhất là [Điều chỉnh]](https://getcomposer.org/doc/06-config.md#sort-packages) trong
`composer.json`:  

    
    
    {
    ...
        "config": {
            "sort-packages": true
        },
    â¦
    }
    
Lần tới, bạn `require` 1 package mới, nó sẽ được thêm vào chỗ thích hợp (và không phải ở cuối).  

## Lời khuyênp #9:  Đừng thử gộp composer.lock khi rebase hay merge  

Nếu bạn thêm 1 dependency mới vào `composer.json` (và `composer.lock`) trước khi nhánh của bạn được gộp, có 1 dependency khác đãđược thêm ở master, bạn cần rebase nhánh của bạn. Và bạn sẽ nhận 1 merge-conflict trong `composer.lock`.  

Bạn đừng bảo giờ cố giải quyết xung đột này thủ công, vì file `composer.lock` chứa từng phần của các dependency đã định nghĩa trong `composer.json`. Vì vậy nếu bạn giải quyết được xung đột, kết quả file lock vẫn sẽ sai.  

Điều tốt nhất có thể làm là tạo `.gitattributes` ở project root với dòng sau, để chỉ ra rằng git sẽ không bao giờ thử gộp `composer.lock`: 
    
    
    /composer.lock -merge
    

Bạn có thể khắc phục vấn đề này bằng cách sử dụng các nhánh tính năng ngắn hạn được gợi ý trong [Trunk Based Development](https://trunkbaseddevelopment.com/) ( bạn nên thực hiện điều này). Khi bạn có 1 nhánh ngắn hạn, nó sẽ được gộp kịp thời, rủi ro của việc xung đột khi gộp trong `composer.lock` là tối thiểu. Bạn thậm chí còn có thể tạo 1 nhánh chỉ cho để thêm 1 dependency và gộp nó đúng cách.  

Nhưng phải làm gì, nếu bạn gặp xung đột gộp ở `composer.lock` khi rebase? Giải quyết điều này với phiên bản từ master, vì vậy bạn sẽ chỉ có những thay đổi trong `composer.json` (những package được thêm mới). Và sau đó chạy `composer update --lock`, nó sẽ cập nhật file `composer.lock `với những thay đổi trong `composer.json`. Bây giờ bạn có thể triển khai file `composer.lock` đã được cập nhật và tiếp tục rebase.  

## Lời khuyên #10: Hiểu sự khác biệt giữa require và require-dev  

Việc nhận thức được sự khác biệt giữa `require` và `require-dev` là thực sự quan trọng.  

Các Package được require để chạy ứng dụng hay thư viện nên được định nghĩa trong `require` (ví dụ: Symfony, Doctrine, Twig, Guzzle, ...) Nếu bạn đang tạo 1 thư viện, hãy cẩn thận với những gì bạn đặt trong `require`. Vì mỗi dependency trong phần này cũng là 1 dependency của ứng dụng mà sử dụng thư viện.  

Các Package cần thiết cho việc phát triển ứng dụng (hoặc thư viện) nên được định nghĩa trong `require-dev` (ví dụ PHPUnit, PHP_codeSniffer, PHPStan).  

## Lời khuyênp #11: Cập nhật các dependency một cách cẩn thận  

Tôi đoán rằng chúng ta đều đồng ý 1 sự thật là các dependency nên được cập nhật thường xuyên. Thứ tôi muốn nói ở đây là việc cập nhật các dependency nên được thực hiện 1 cách rõ ràng và thận trọng, chứ không phải chỉ là làm cho xong như các việc khác. Nếu bạn tái cấu trúc 1 cái gì đó và tại cùng thời điểm với việc cập nhật 1 vài thư viện, bạn không thể dễ dàng biết là ứng dụng bị hỏng do việc tái cấu trúc hay do việc cập nhật.  

Bạn có thể sử dụng câu lệnh `composer oudated` để xem các dependency có thể  cập nhật. Bạn nên thêm lựa chọn `--direct` (hay `-D`) để chỉ hiển thị danh sách các dependency được khai báo trong `composer.json`. Ngoài ra cũng có lựa chọn `-m` để chỉ hiển thị các bản cập nhật nhỏ.  

**Với mỗi dependency quá hạn thì thực hiện các bước sau đây:**

  1. Tạo một nhánh mới
  2. Cập nhật phiên bản phụ thuộc trong `composer.json` lên phiên bản mới nhất.
  3. Chạy lệnh `composer update phpunit/phpunit --with-dependencies` (thay `phpunit/phpunit` bằng thư viện bạn đang update)
  4. Kiểm tra CHANGELOG trong repo thư viện trên Github để xem có sự thay đổi nào hỏng không. Nếu có cập nhật ứng dụng.
  5. Kiểm tra thử ứng dụng trên local (Nếu đang sử dụng Symfony, bạn có thể thấy các cảnh báo phản đối trong Debug Bar)
  6. Commit các thay đổi (`composer.json`, `composer.lock` và bất cứ thứ gì cần thiết để phiên bản mới hoạ động)
  7. Đợi kiến trúc CI hoàn thành
  8. Merge và deploy

Đôi khi sẽ xảy ra vấn đề khi cập nhật nhiều phụ thuộc cùng lúc, ví dụ như khi bạn cập nhật Doctrine hay Symfony. Trường hợp này bạn có thể liệt kê tất cả trong câu lệnh update.  


    composer update symfony/symfony symfony/monolog-bundle --with-dependencies
    

Hoặc bạn cũng có thể sử dụng wildcard để cập nhật tất cả các phụ thuộc theo 1 namespace cụ thể:  
    
    composer update symfony/* --with-dependencies
    
Tôi biết rằng tất cả những điều này nghe có vẻ thừa thãi, nhưng bạn chắc chắn sẽ cập nhật các phụ thuộc 1 cách tình cờ, an toàn hơn vẫn đáng.  

Một các ngắn gọn mà mọi người chấp nhận thực hiện là cập nhật tất cả các dependency `require-dev` cùng lúc (nếu chúng không yêu cầu những thay đổi bên trong code, mặt khác tôi khuyên bạn nên sử dụng các nhánh riêng biệt để dễ dàng review code).  

## Lời khuyên #12:  Bạn có thể định nghĩa các loại dependency khác nhau trong `composer.json`
Ngoài việc định nghĩa các thư viện như là các dependency , bạn cũng có thể định nghĩa các thứ khác ở đây.  
Bạn có thể định nghĩa phiên bản PHP mà ứng dụng/thư viện của bạn hỗ trợ:  
    
    "require": {
        "php": "7.1.* || 7.2.*",
    },
    

Bạn cũng có thể định nghĩa các extensions được yêu cầu trong ứng dụng/thư viện. Điều này rất hữu ích khi bạn cố gắng dockerize ứng dụng của bạn hay khi đồng nghiệp mới của bạn cài đặt ứng dụng lần đầu tiên.  
    
    "require": {
        "ext-mbstring": "*",
        "ext-pdo_mysql": "*",
    },
    

(Bạn nên dùng `*` cho phiên bản của extensions [Chúng có thể không nhất quán](https://getcomposer.org/doc/01-basic-usage.md#platform-packages)).  

## Lời khuyên #13: Xác thực `composer.json` trong quá trình CI build  

`composer.json` và `composer.lock` nên luôn đồng bộ. vì thế, sẽ là ý tưởng hay nếu có kiêm tra tự động. Chỉ cần thêm đoạn sau vào 1 phần của đoạn mã lệnh xây dựng và nó sẽ đảm bảo `composer.lock` đồng bộ với `composer.json`)  
    
    composer validate --no-check-all --strict
    

## Lời khuyên #14: Dùng Composer plugin trong PHPStorm  

Có [composer.json plugin echo PHPStorm](https://plugins.jetbrains.com/plugin/7631-php-composer-json-support). Nó thêm tự động hoàn thiện và xác thực khi thay đổi`composer.json` thủ công.  
Nếu bạn đang dùng ide khác (hay code editor), bạn có thể thiết lập [JSON schema](https://getcomposer.org/schema.json).  

## Lời khuyên #15: Specify the production PHP version in `composer.json`  

Nếu bạn giống tôi và đổi khi [chạy các phiên bản pre-released PHP trên local](https://blog.martinhujer.cz/php-7-2-is-due-in-november-whats-new/), bạn đang gặp vấn đề trong việc cập nhật các dependency lên 1 phiên bản không hoạt động trong sản phẩm. Ngy bây giờ tôi đang sử dụng PHP 7.2.0, có nghĩa là tôi có thể cài đặt các thư viện, mà không hoạt động trên 7.1. Vì sản phẩm đang chạy 7.1, quá trình cài đặt sẽ thất bại.  

Nhưng bạn không cần lo lắng, có cách giải quyết dễ dàng. Chỉ cần định nghĩa phiên bản production PHP trong phần `config` của `composer.jon`  
    
    "config": {
        "platform": {
            "php": "7.1"
        }
    }
    
Đừng nhầm lẫn nó với `require`, nó khác nhau hoàn toàn. Ứng dụng của bạn có thể chạy trên cả 7.1 hay 7.2 và cùng lúc định nghĩa 7.1 như 1 nền tảng (có nghĩa là các dependency luôn cập nhật đến 1 phiên bản tương thích với 7.1) 
    
    "require": {
        "php": "7.1.* || 7.2.*"
    },
    "config": {
        "platform": {
            "php": "7.1"
        }
    },
    

## Lời khuyên #16: Dùng các package riêng tư từ self-hosted Gitlab  

Nên sử dụng `vcs` như là 1 loại repository và Composer sẽ quyết định cách đúng để lấy các package đó. Ví dụ, nếu bạn thêm 1 fork từ Github, nó sẽ sử dụng API của nó để tải file .zip thay vì clone toàn bọ repository.  

Nhưng điều này sẽ phức tạp hợp khi cài đặt riêng tư trên Gitlab. Nếu bạn sử dụng `vcs` như 1 loại repository, Composer sẽ phát hiện ra đây là 1 cài đặt Gitlab cố gắng tải package bằng cách sử dụng API (được yêu cầu API key. Tôi không muốn cài đặt nó, vì vậy tôi cài đặt thiết lập này(sử dụng SSH để clone))    

Trước tiên định nghĩa repository với loại `git`:  

    
    "repositories": [
        {
            "type": "git",
            "url": "git@gitlab.mycompany.cz:package-namespace/package-name.git"
        }
    ]
    

Sau đó dùng package như bạn làm thủ công:  

    
    
    "require": {
        "package-namespace/package-name": "1.0.0"
    }
    

## Lời khuyên #17: Làm sao để tạm thời sử dụng 1 nhánh với bugfix từ fork  

Nếu bạn tìm ra bg trong vài thư viện được public và sủa nó trong bản fork trên github, bạn cần cài thư viện từ repo này thay vì bản official (cho đến khi bugfix được hợp và bản đã sửa được phát hành).  

Ns có thể được thực hiện dễ dàng với [inline aliasing](https://getcomposer.org/doc/articles/aliases.md#require-inline-alias):  

    
    {
        "repositories": [
            {
                "type": "vcs",
                "url": "https://github.com/you/monolog"
            }
        ],
        "require": {
            "symfony/monolog-bundle": "2.0",
            "monolog/monolog": "dev-bugfix as 1.0.x-dev"
        }
    }
    

Bản có thể test fixbug trên local trước khi đẩy bằng cách [dùng `path` như là một kiểu repo](https://getcomposer.org/doc/05-repositories.md#path).

## Update 2018-01-08:
Sau khi phát hành bài viết, tôi nhận được vài gợi ý thêm một vài tip. Vậy chúng đây:

## Lời khuyên #18:  Cài đặt prestissimo để tăng tốc độ cài đặt package

Composer plugin [hirak/prestissimo](https://github.com/hirak/prestissimo) sẽ tăng tốc độ cài dependency bằng cách download song song.  
VÀ điều tốt nhất là? bạn chỉ cần cái 1 lần, trên toàn cục và nó sẽ hoạt độn trên mọi project  
    
    composer global require hirak/prestissimo
    

## Lời khuyên #19:  Kiểm tra ràng buộc phiên bản của bạn nếu cảm thấy không chắc chắn  

Viết đúng ràng buộc phiên bản có thể là 1 việc khó khăn ngay cả sau khi đọc [tài liệu](https://getcomposer.org/doc/articles/versions.md#writing-version-constraints).  

May măn là có [Packagist Semver Checker](https://semver.mwl.be/), bạn có thể kiểm tra phiên bản nào phù hợp với ràng buộc được khai báo. Thay vì chỉ phân tích ràng buộc phiên bản, nó tải dữ liệu từ Packagist để hiển thị phiên bản được realease hiện tại. 

[the result for `symfony/symfony:^3.1`](https://semver.mwl.be/#?package=symfony%2Fsymfony&version=%5E3.1&minimum-
stability=stable).

## Lời khuyên #20: Dùng authoritative class map trong production  

Bạn nên [taoj authoritative class map](https://getcomposer.org/doc/articles/autoloader-optimization.md#optimization-level-2-a-authoritative-class-maps) trong
production. Nó sẽ tăng tốc load class bằng cách thêm moitj thứ trong class-map và bỏ qua kiểm tra filesystem.

bạn có thể thực hiện nó bằng cách chạy lệnh sau như một phần của quá trình xây dựng sản phẩm

    
    composer dump-autoload --classmap-authoritative
    

## Lời khuyên #21: Cấu hình `autoload-dev` để tests  

Bạn thường không muốn thêm các file test trong production class map (cì kích thước file và bộ nhớ). Nó có thể thực hiên bằng cách cấu hình `autoload-dev` (tương tự như `autoload`):  
    
    
    "autoload": {
        "psr-4": {
            "Acme\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Acme\\": "tests/"
        }
    },
    

## Lời khuyên #22: Thử Composer scripts  

Composer scripts là công cụ nhẹ để tạo build script. Tôi đã viêt [bài viết rieeng về chúng](/have-you-tried-composer-scripts/).
