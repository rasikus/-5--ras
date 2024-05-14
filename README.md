using System;
using System.Reflection.Metadata;

namespace MyApp // Note: actual namespace depends on the project name.
{
    internal class Program
    {
        public enum AttackType
        {
            Damage,
            Self,
            Heal
        
        }
        static void Main(string[] args)
        {
            string? name;
            float maxPlayerHealth = 1000f;
            float maxEnemyHealth = 2000f;

            float currentPlayerHealth = maxPlayerHealth;
            float currentEnemyHealth = maxEnemyHealth;

            float playerDamage = 50f;
            float enemyDamage = 55f;

            float fireballDamage = 200f;
            float enemyHealDamage = 100f;

            string? input;

            int commandCount = 3;
            int enemySecondAbilityModifer = 3;

            bool BlowWeaponEnemy = true;                                     // Создание переменной удара противника оружием или нет
            int BlowWeaponEnemyActionCount = 100;

            Random randomizer = new Random();

            Dictionary<AttackType, List<float>> damagesToPlayer = new();  
            Dictionary<AttackType, List<float>> damagesToEnemy = new();

            damagesToPlayer[AttackType.Damage] = new List<float>();
            damagesToPlayer[AttackType.Self] = new List<float>();
            damagesToPlayer[AttackType.Heal] = new List<float>();

            damagesToEnemy[AttackType.Damage] = new List<float>();
            damagesToEnemy[AttackType.Self] = new List<float>();
            damagesToEnemy[AttackType.Heal] = new List<float>();

            HelloName();                                                   //Создаем метод 

            void HelloName() 
            {
                Console.WriteLine("Введите ваше имя: ");
                name = Console.ReadLine();

                Console.Clear();
                Console.WriteLine($"Добро пожаловать в игру, {name}\n");
            }

            while (currentEnemyHealth > 0 && currentPlayerHealth > 0)
            {
                HealthShower();                                                    // Меняем цвет вывода здоровья

                void HealthShower() 
                {
                    if (currentPlayerHealth > 100)
                        Console.ForegroundColor = ConsoleColor.Green;
                    else
                        Console.ForegroundColor = ConsoleColor.Red;
                    Console.WriteLine($"Ваше здоровье: {currentPlayerHealth}");
                    Console.ResetColor();

                    if (currentEnemyHealth > 200)
                        Console.ForegroundColor = ConsoleColor.Green;
                    else
                        Console.ForegroundColor = ConsoleColor.Red;
                    Console.WriteLine($"Здоровье противника: {currentEnemyHealth}");
                    Console.ResetColor();
                }              

                ChosingAction();                                                      //Создаем метод выбора действия

                void ChosingAction() 
                {
                    Console.WriteLine($"{name}! Выберите действие:\n" +
                    $"1. Ударить оружием(урон {playerDamage})\n" +
                    $"2. Щит: следующая атака противника не наносит урона\n" +
                    $"3. Огненный шар: наносит урон в размере {fireballDamage}\n");
                }


                input = Console.ReadLine();
                bool playerShield = false;
                bool playerDamageWithWeapon = false;
                switch (input) 
                {
                    case"1":
                        playerDamageWithWeapon = true;
                        currentEnemyHealth -= playerDamage;
                        damagesToEnemy[AttackType.Damage].Add(playerDamage);
                        break;
                    case"2":
                        playerShield = true;
                        break;
                    case"3":
                        currentEnemyHealth -= fireballDamage;
                        damagesToEnemy[AttackType.Damage].Add(fireballDamage);
                        break;
                    default:
                        Console.WriteLine("Команда не распознана!");
                        break;
                }
                int enemyCommand = randomizer.Next(1, commandCount+1);
                switch (enemyCommand)
                {
                    case 1:
                        BlowWeaponEnemy = true;
                        if (!playerShield)
                        {
                            currentPlayerHealth -= enemyDamage;
                            damagesToPlayer[AttackType.Damage].Add(enemyDamage);
                        }
                        break;
                    case 2:
                        BlowWeaponEnemy = true;
                        if (!playerShield)
                        {
                            currentEnemyHealth -= enemyDamage;
                            currentPlayerHealth -= enemyDamage * enemySecondAbilityModifer;
                            damagesToEnemy[AttackType.Self].Add(enemyDamage);
                            damagesToPlayer[AttackType.Damage].Add(enemyDamage* enemySecondAbilityModifer);
                        }
                        break;
                    case 3:
                        BlowWeaponEnemy = false;
                        if (playerDamageWithWeapon)
                        {
                            currentEnemyHealth -= enemyHealDamage;
                            damagesToEnemy[AttackType.Self].Add(enemyHealDamage);
                        }
                        else
                        {
                            currentEnemyHealth += enemyHealDamage;
                            damagesToPlayer[AttackType.Heal].Add(enemyHealDamage);
                            if(currentEnemyHealth > maxEnemyHealth)
                                currentEnemyHealth = maxEnemyHealth;
                        }
                        break;

                }

                BlowWeaponEnemyAction();

                void BlowWeaponEnemyAction()
                {
                    if (BlowWeaponEnemy == false)                                                  //Созданный Метод при не оружием
                    {
                        currentPlayerHealth += BlowWeaponEnemyActionCount;
                        damagesToEnemy[AttackType.Heal].Add(BlowWeaponEnemyActionCount);
                    }
                    if (currentPlayerHealth > maxPlayerHealth)
                        currentPlayerHealth = maxPlayerHealth;


                }

                WriteDamagesFromTo(damagesToEnemy, name, "Enemy");
                WriteDamagesFromTo(damagesToPlayer,"Enemy",name);

                EndTurn();

                void EndTurn() 
                {
                    Console.WriteLine("\nДля продолжения нажмите любую клавишу.");
                    Console.ReadKey();
                    Console.Clear();

                }                

                void DamageMessage (string damager, string defender, float damage) 
                {
                    Console.ForegroundColor = ConsoleColor.Red;                                                        //Смена цвета вывода урона
                    Console.WriteLine($"Игрок {damager} наносит игроку {defender} урон в размере {damage} едениц.");
                    Console.ResetColor();
                }
               
                void HealMessage(string healer, float healValue)
                {
                    Console.WriteLine($"Игрок {healer} восстановил себе здоровье в размере {healValue} едениц.");
                
                }
                void WriteDamageWithType(AttackType attackType, string damageName, string defenderName, List<float> damages) 
                { 
                    foreach (var damage in damages) 
                    {
                        switch (attackType)
                        {
                            case AttackType.Damage:
                                DamageMessage(damageName, defenderName, damage);
                                break;
                            case AttackType.Self:
                                DamageMessage(defenderName, defenderName, damage);
                                break;
                            case AttackType.Heal:
                                HealMessage(damageName, damage);
                                break;
                            default:
                                break;
                        }
                    }
                    damages.Clear();

                }

                void WriteDamagesFromTo(Dictionary<AttackType, List<float>> damages, string damagerName, string defenderName)
                {
                    foreach(var damage in damages)
                    {
                        WriteDamageWithType(damage.Key, damagerName, defenderName, damage.Value);
                    }
                }

            }
            if (currentPlayerHealth > currentEnemyHealth)   // Моя приписка
            {
                Console.WriteLine("Победа!");
            }
            else 
            {
                Console.WriteLine("Поражение.");
            }
        }
    }
}
