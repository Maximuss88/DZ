������ 1.
������� ������������� ������ �������� ��� AWS ����������, ������� ����� ������������ ������: https://github.com/hashicorp/terraform-provider-aws.git. ������ ������� ������ ������� � �������� ���� � ������ �� ������� ������ �������.
    �������, ��� ����������� ��� ��������� resource � data_source, ��������� ������ �� ��� ������ � ���� �� �������.
    ��� �������� ������� ��������� SQS ������������ ������ aws_sqs_queue � �������� ���� �������� name.
        � ����� ������ ���������� ����������� name? ��������� ������� ����, � ������� ��� �������.
        ����� ������������ ����� �����?
        ������ ����������� ��������� ������ ����������� ���?
        

���� � ��������� ����� (����������� ����������� � grep �� �����������), ��:

�) ��� ����� https://github.com/hashicorp/terraform-provider-aws/blob/main/aws/data_source_aws_sqs_queue_test.go � https://github.com/hashicorp/terraform-provider-aws/blob/main/aws/resource_aws_sqs_queue_test.go

�) � name_prefix (Conflicts with `name_prefix`. * `name_prefix` - (Optional) Creates a unique name beginning with the specified prefix. Conflicts with `name`)

�) 80 �������� (and must be between 1 and 80 characters long)

�) For a FIFO (first-in-first-out) queue, the name must end with the `.fifo` suffix.

