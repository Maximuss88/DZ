«адача 1.
ƒавайте потренируемс€ читать исходный код AWS провайдера, который можно склонировать отсюда: https://github.com/hashicorp/terraform-provider-aws.git. ѕросто найдите нужные ресурсы в исходном коде и ответы на вопросы станут пон€тны.
    Ќайдите, где перечислены все доступные resource и data_source, приложите ссылку на эти строки в коде на гитхабе.
    ƒл€ создани€ очереди сообщений SQS используетс€ ресурс aws_sqs_queue у которого есть параметр name.
        — каким другим параметром конфликтует name? ѕриложите строчку кода, в которой это указано.
         ака€ максимальна€ длина имени?
         акому регул€рному выражению должно подчин€тьс€ им€?
        

≈сли € правильно нашел (склонировал репозиторий и grep по содержимому), то:

а) это файлы https://github.com/hashicorp/terraform-provider-aws/blob/main/aws/data_source_aws_sqs_queue_test.go и https://github.com/hashicorp/terraform-provider-aws/blob/main/aws/resource_aws_sqs_queue_test.go

б) с name_prefix (Conflicts with `name_prefix`. * `name_prefix` - (Optional) Creates a unique name beginning with the specified prefix. Conflicts with `name`)

в) 80 символов (and must be between 1 and 80 characters long)

г) For a FIFO (first-in-first-out) queue, the name must end with the `.fifo` suffix.

