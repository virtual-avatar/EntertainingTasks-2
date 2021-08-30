# EntertainingTasks-2
```
<?php
/*
    Необходимо доработать класс рассылки Newsletter, что бы он отправлял письма
    и пуш нотификации для юзеров из UserRepository.

    За отправку имейла мы считаем вывод в консоль строки: "Email {email} has been sent to user {name}"
    За отправку пуш нотификации: "Push notification has been sent to user {name} with device_id {device_id}"

    Так же необходимо реализовать функциональность для валидации имейлов/пушей:
    1) Нельзя отправлять письма юзерам с невалидными имейлами
    2) Нельзя отправлять пуши юзерам с невалидными device_id. Правила валидации можете придумать сами.
    3) Ничего не отправляем юзерам у которых нет имен
    4) На одно и то же мыло/device_id - можно отправить письмо/пуш только один раз

    Для обеспечения возможности масштабирования системы (добавление новых типов отправок и новых валидаторов),
    можно добавлять и использовать новые классы и другие языковые конструкции php в любом количестве.
    Реализация должна соответствовать принципам ООП
*/
abstract class AbstractNewsletter
{
    abstract public function createNewsletter(): NewsletterInterface;

    public function sendNewsletter(): void
    {
        $newsletter = $this->createNewsletter();
        $newsletter->send();
    }
}

class NewsletterEmail extends AbstractNewsletter
{
    public function createNewsletter(): NewsletterInterface
    {
        return new NewsletterEmailSender();
    }
}

class NewsletterPush extends AbstractNewsletter
{
    public function createNewsletter(): NewsletterInterface
    {
        return new NewsletterPushSender();
    }
}

trait UsersList {
    private function getUsersList():array
    {
        $validator = new Validator();
        $users = new UserRepository();
        return $validator->validateUsersList($users->getUsers());
    }
}

interface NewsletterInterface
{
    public function send(): void;
}

class NewsletterEmailSender implements NewsletterInterface
{
    use UsersList;

    public function send(): void
    {
        $users = $this->getUsersList();
        foreach ( $users as $user) {
            if(!empty($user['email'])) {
                echo "Email {$user['email']} has been sent to user {$user['name']}\n";
            }
        }
    }
}

class NewsletterPushSender implements NewsletterInterface
{
    use UsersList;

    public function send(): void
    {
        $users = $this->getUsersList();
        foreach ( $users as $user) {
            if(!empty($user['device_id']))
            echo "Push notification has been sent to user {$user['name']} with device_id {$user['device_id']}\n";
        }
    }
}

class Validator
{
    /**
     * Требование - Нельзя отправлять письма юзерам с невалидными имейлами
     * Решение - если имейл некоррентный возвращаем false
     * @param string|null $email
     * @return bool
     */
    protected function isValidEmail(?string $email): bool
    {
        if (filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return true;
        } else {
            return false;
        }
    }

    /**
     * Требование - Нельзя отправлять пуши юзерам с невалидными device_id.
     * Решение - если device_id невалидный возвращаем false
     * @param string|null $deviceId
     * @return bool
     */
    protected function isValidDeviceId(?string $deviceId): bool
    {
        if (!empty($deviceId)) {
            return true;
        } else {
            return false;
        }
    }

    /**
     * Требование - Ничего не отправляем юзерам у которых нет имен
     * Решение - если имени нет возвращаем false
     * @param string|null $name
     * @return bool
     */
    protected function isValidName(?string $name): bool
    {
        if (!empty($name)) {
            return true;
        } else {
            return false;
        }
    }

    /**
     * Требование - На одно и то же мыло/device_id - можно отправить письмо/пуш только один раз
     * Решение - удаляем из массива не унакальные данные, так как доп.требований нет, удаляем первые найденные
     * @param array $users
     * @return array
     */
    protected function sanitizeUsersList(array $users): array
    {
        $emailArray = array_count_values(array_column($users,'email'));
        $deviceIdArray = array_count_values(array_column($users,'device_id'));
        foreach ($users as &$user) {
            if(!empty($user['email']) && $emailArray[$user['email']] > 1) {
                $emailArray[$user['email']]--;
                unset($user['email']);
            }
            if(!empty($user['device_id']) && $deviceIdArray[$user['device_id']] > 1) {
                $deviceIdArray[$user['device_id']]--;
                unset($user['device_id']);
            }
        }
        return $users;
    }

    /**
     * Возвращаем провалидированный массив данных пользователей
     * @param array $users
     * @return array
     */
    public function validateUsersList(array $users): array
    {
        $usersList = $this->sanitizeUsersList($users);

        $validatedList = [];
        foreach ($usersList as $user) {
            $userName = empty($user['name']) ? null : $user['name'];
            if(!$this->isValidName($userName)) {
                continue;
            }
            $userEmail = empty($user['email']) ? null : $user['email'];
            if(!$this->isValidEmail($userEmail)) {
                unset($user['email']);
            }
            $userDeviceId = empty($user['device_id']) ? null : $user['device_id'];
            if(!$this->isValidDeviceId($userDeviceId)) {
                unset($user['device_id']);
            }
            if(!empty($userEmail) || !empty($userDeviceId)) {
                $validatedList[] = $user;
            } else {
                continue;
            }
        }
        return $validatedList;
    }

}

class UserRepository
{
    public function getUsers(): array
    {
        return [
            [
                'name' => 'Ivan',
                'email' => 'ivan@test.com',
                'device_id' => 'Ks[dqweer4'
            ],
            [
                'name' => 'Ivan1',
                'email' => 'ivan@test.com'
            ],
            [
                'name' => 'Peter',
                'email' => 'peter@test.com'
            ],
            [
                'name' => 'Mark',
                'device_id' => 'Ks[dqweer4'
            ],
            [
                'name' => 'Nina',
                'email' => '...'
            ],
            [
                'name' => 'Luke',
                'device_id' => 'vfehlfg43g'
            ],
            [
                'name' => 'Zerg',
                'device_id' => ''
            ],
            [
                'email' => '...',
                'device_id' => ''
            ]
        ];
    }
}

/**
 * Тут релизовать получение объекта(ов) рассылки Newsletter и вызов(ы) метода send()
 * $newsletter = //... TODO
 * $newsletter->send();
 */

/**
 * @param AbstractNewsletter $newsletter
 */
function sendAnyNewsletter(AbstractNewsletter $newsletter):void
{
    $newsletter->sendNewsletter();
}

sendAnyNewsletter(new NewsletterEmail());
sendAnyNewsletter(new NewsletterPush());
```
