# skrepicoco
lecture 7
pragma solidity 0.8.0;

import "./BokkyPooBahsDateTimeLibrary.sol";

contract BirthdayPayout {
    string private _name;                               // Приватная переменная для хранения названия контракта
    address private _owner;                             // Приватная переменная для хранения адреса владельца контракта
    mapping(address => bool) private _receivedGifts;    // Маппинг для отслеживания получили ли сотрудники подарок
    Teammate[] private _teammates;                       // Массив для хранения информации о сотрудниках

    struct Teammate {
        string name;                                    // Имя сотрудника
        address account;                                // Ethereum адрес сотрудника
        uint256 salary;                                 // Сумма зарплаты для выплаты
        uint256 birthday;                               // Таймстамп дня рождения сотрудника
    }

    constructor() {
        _name = "max";
        _owner = msg.sender;
    }

    function addTeammate(address account, string memory name, uint256 salary, uint256 birthday) public onlyOwner {
        require(msg.sender != account, "Cannot add oneself");
        Teammate memory newTeammate = Teammate(name, account, salary, birthday);  // Создание новой структуры сотрудника
        _teammates.push(newTeammate);                     // Добавляем нового сотрудника в массив
        emit NewTeammate(account, name);                  // Уведомления о добавлении нового сотрудника
    }

    function birthdayPayout() public onlyOwner {
        for (uint256 i = 0; i < _teammates.length; i++) {
            if (checkBirthday(i) && !_receivedGifts[_teammates[i].account]) {
                sendToTeammate(i);
                _receivedGifts[_teammates[i].account] = true; // Установка флага, что сотрудник получил подарок
                emit HappyBirthday(_teammates[i].name, _teammates[i].account); // уведомления о подарке на день рождения
        }
            }
        }
    }

    function getDate(uint256 timestamp) public view returns (uint256 year, uint256 month, uint256 day) {
        (year, month, day) = BokkyPooBahsDateTimeLibrary.timestampToDate(timestamp);  // Использование внешней библиотеки для преобразования таймстампа в дату
    }

    function checkBirthday(uint256 index) public view returns (bool) {
        uint256 birthday = _teammates[index].birthday;    // Получение таймстампа дня рождения сотрудника
        (, uint256 birthday_month, uint256 birthday_day) = getDate(birthday);  // Получение месяца и дня дня рождения сотрудника
        uint256 today = block.timestamp;                   // Получение текущего таймстампа
        (, uint256 today_month, uint256 today_day) = getDate(today);  // Получение месяца и дня текущей даты

        return (birthday_day == today_day && birthday_month == today_month);  // Проверка, совпадает ли день рождения с текущей датой
    }

    function getTeammate(uint256 index) public view returns (Teammate memory) {
        return _teammates[index];                          // Получение информации о сотруднике по индексу
    }

    function getTeam() public view returns (Teammate[] memory) {
        return _teammates;                                  // Получение всех сотрудников
    }

    function getTeammatesNumber() public view returns (uint256) {
        return _teammates.length;                           // Получение количества сотрудников
    }

    function sendToTeammate(uint256 index) private {
        payable(_teammates[index].account).transfer(_teammates[index].salary);  // Отправка зарплаты сотруднику
    }

    function deposit() public payable {}

    modifier onlyOwner {
        require(msg.sender == _owner, "Sender should be the owner of the contract");  // Модификатор для выполнения функций только владельцем контракта
        _;
    }

    event NewTeammate(address account, string name);        // Событие для уведомления о добавлении нового сотрудника
    event HappyBirthday(string name, address account);      // Событие для уведомления о подарке сотруднику на день рождения
}
