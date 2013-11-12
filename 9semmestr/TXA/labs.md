# 1 Создание модульных тестов для проекта

Срок сдачи: 7 марта 2013 г.

Эта возможность встроена в популярные среды разработки, такие как Visual Studio и Eclipse.

**Задание: создать не менее пяти осмысленных модульных тестов для проекта и добиться их успешного выполнения. Тесты должны включать**

* хотя бы один тест на исключение (проверка того, что метод действительно возбуждает исключение определённого типа при возникновении исключительной ситуации);
* хотя бы один тест, проверяющий возвращаемое значение, являющееся коллекцией;
* хотя бы один тест для метода, не возвращающего значение (void).

**Убедитесь, что включили модульные тесты в репозитарий системы управления версиями, так как они потребуются в дальнейшем при настройке системы непрерывной интеграции.**



# 2 Сборка проекта в командной строке

Срок сдачи: 6 апреля 2013 г.

## Определение расположения утилиты **MSBuild.exe**

Чтобы определить местоположение **MSBuild.exe** в системе, запустите **Visual Studio Command Prompt** и выполните команду

    where msbuild

Эта команда выведет список всех расположений утилиты **MSBuild.exe**, например:

	C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe
	C:\Windows\Microsoft.NET\Framework\v3.5\MSBuild.exe

## Сборка проекта

Находясь в папке проекта, выполнить команду

    c:\windows\microsoft.net\framework\v4.0.30319\msbuild.exe

(для .NET Framework v4.0.30319).


**Задание: cобрать проект (получить исполняемый файл или установочный пакет) из командной строки.**


## Построение и выполнение модульных тестов

Для выполнения модульных тестов используется утилита **MSTest.exe**. Определить её расположение можно используя команду `where`:

    > where mstest
    c:\Program Files (x86)\Microsoft Visual Studio 10.0\Common7\IDE\MSTest.exe

Для выполения тестов необходимо предварительно выполнить сборку проекта, содержащего тесты (допустим, он называется "TestProject"):

    msbuild "TestProject\TestProject.csproj"


Команда для выполнения тестов:

    mstest "/testcontainer:TestProject\bin\Debug\TestProject.dll"
    

**Задание: выполнить модульные тесты из командной строки.**



# 3 Сборка и тестирование проекта с помощью системы автоматического построения

Срок сдачи: 4 мая 2013 г.

Для того, чтобы выполнять построение проекта и его тестирование с помощью одной команды, необходим конфигурационный файл MSBuild. Пример такого файла:
	
	<?xml version="1.0" encoding="utf-8" ?>
	<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003" DefaultTargets="Test">
	  <ItemGroup>
	    <BuildArtifactsDir Include="BuildArtifacts\" />
	    <SolutionFile Include="Project.sln" />
	  </ItemGroup>
	
	  <PropertyGroup>
	    <Configuration Condition=" '$(Configuration)' == '' ">Release</Configuration>
	    <BuildPlatform Condition=" '$(BuildPlatform)' == '' ">Any CPU</BuildPlatform>
	  </PropertyGroup>
	
	  <Target Name="Init" DependsOnTargets="Clean">
	    <MakeDir Directories="@(BuildArtifactsDir)" />
	  </Target>
	
	  <Target Name="Clean">
	    <RemoveDir Directories="@(BuildArtifactsDir)" />
	  </Target>
	
	  <Target Name="Compile" DependsOnTargets="Init">
	    <MSBuild Projects="@(SolutionFile)" Targets="Rebuild"
	             Properties="OutDir=%(BuildArtifactsDir.FullPath);Configuration=$(Configuration);Platform=$(BuildPlatform)" />
	  </Target>
	
	  <Target Name="Test" DependsOnTargets="Compile">
	    <PropertyGroup>
	      <TestSuccess>1</TestSuccess>
	    </PropertyGroup>
	    <Exec Command='"$(VS100COMNTOOLS)..\IDE\mstest.exe" /testcontainer:@(BuildArtifactsDir)\TestProject.dll' >
	      <Output TaskParameter="ExitCode" PropertyName="TestSuccess"/>
	    </Exec>
	  </Target>
	</Project>

[Источник](http://www.infoq.com/articles/MSBuild-1)


Запуск построения:

    msbuild Project.msbuild

Результат:

    ...
	Build succeeded.
	    0 Warning(s)
	    0 Error(s)
	
	Time Elapsed 00:00:03.36
	
**Задание: разработать конфигурационный файл MSBuild для сборки и тестирования своего проекта и проверить его работу в командной строке.**

**Убедитесь, что включили конфигурационный файл в репозитарий системы управления версиями, так как он потребуется при настройке системы непрерывной интеграции.**



# 4 Настройка системы непрерывной интеграции

Срок сдачи: 1 июня 2013 г.

1. Войдите в систему непрерывной интеграции по адресу http://ic-osu-204-1:8080/ (доступен только из КЦ-204) и создайте учётную запись.
2. Создайте новую задачу. В поле **Job name** введите свои фамилию и имя.
3. В настройках задачи в группе **Source Code Management** выберите свою ситему управления версиями и укажите ссылку на свой репозитарий.
4. В группе **Build Triggers** установите опцию **Poll SCM**.
5. В группе **Build** добавьте шаг **Build a Visual Studio project or solution using MSBuild** и введите имя конфигурационного файла в поле **MSBuild Build File**.
6. Сохраните задачу.
7. Запустите построение (**Build Now**).
8. Откройте страницу **Console Output** и удостоверьтесь, что построение было выполнено успешно, и были запущены все требуемые шаги, включая тестирование проекта.
9. Теперь при изменении исходного кода в системе управления версиями Jenkins будет выполнять сборку и тестирование проекта.

